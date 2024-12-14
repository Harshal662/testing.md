When handling multiple occurrences of the same `BILLING_CODE` in the `nahRulesMap`, you need to account for situations where a `BILLING_CODE` appears more than once with different values in various columns. To handle this scenario, you can modify the `loadNahRules` function to store these multiple entries under the same `BILLING_CODE` key. Specifically, you could store each set of column values for the same `BILLING_CODE` in a list of maps, so that when you need to compare values, you can iterate over all occurrences of that `BILLING_CODE`.

### Updated `loadNahRules` Function

We'll change the `nahRulesMap` from being a map of `String -> Map<String, String>` to a map of `String -> List<Map<String, String>>`. This will allow us to store multiple occurrences of the same `BILLING_CODE`.

### Code Implementation:

```java
private static Map<String, List<Map<String, String>>> loadNahRules(Sheet sheet) {
    Map<String, List<Map<String, String>>> nahRulesMap = new HashMap<>();
    Row headerRow = sheet.getRow(0);
    int billingCodeIndex = findColumnIndex(headerRow, "BILLING_CODE");

    for (int i = 1; i <= sheet.getLastRowNum(); i++) {
        Row row = sheet.getRow(i);
        if (row == null) continue;

        String billingCode = getCellValue(row.getCell(billingCodeIndex));
        if (billingCode == null || billingCode.isEmpty()) continue;

        Map<String, String> columnValues = new HashMap<>();
        for (int j = 0; j < row.getLastCellNum(); j++) {
            if (j == billingCodeIndex) continue; // Skip the BILLING_CODE column
            String header = getCellValue(headerRow.getCell(j));
            String value = getCellValue(row.getCell(j));
            columnValues.put(header, value);
        }

        // Add this entry to the list of billingCode entries
        if (!nahRulesMap.containsKey(billingCode)) {
            nahRulesMap.put(billingCode, new ArrayList<>());
        }
        nahRulesMap.get(billingCode).add(columnValues);
    }
    return nahRulesMap;
}
```

### Explanation of Changes:

1. **`nahRulesMap` Change**:
   - Instead of mapping `BILLING_CODE` to a single `Map<String, String>`, we map it to a `List<Map<String, String>>`. This allows multiple entries for the same `BILLING_CODE`, each stored as a separate map.

2. **Handling Multiple Entries for the Same `BILLING_CODE`**:
   - When encountering a `BILLING_CODE`, if it already exists in the map, we append the new set of column values to the list associated with that `BILLING_CODE`. If it doesn't exist yet, we initialize a new list and add the first entry.

### Adjustments in `compareWithNahRules` Function:

Now that `nahRulesMap` stores multiple occurrences of the same `BILLING_CODE`, you need to adjust the comparison logic to handle the list of `Map<String, String>` for each `BILLING_CODE`. For each `outputRow`, you will need to compare the values in all entries for the corresponding `BILLING_CODE`.

Here's how to adjust the `compareWithNahRules` function:

1. **Iterate Over All `BILLING_CODE` Entries**: For each `outputRow`, if the `BILLING_CODE` has multiple entries in the `nahRulesMap`, iterate over all of them and compare each set of column values.

2. **Comparison Logic**: If any one of the entries matches (based on your custom matching rules), you can consider that `BILLING_CODE` as a match for that row.

### Updated `compareWithNahRules` Function:

```java
private static List<Map<String, String>> compareWithNahRules(List<Map<String, String>> outputData, Map<String, List<Map<String, String>>> nahRulesMap) {
    List<Map<String, String>> results = new ArrayList<>();

    // Define the relevant columns for comparison (left side - output file, right side - nah rules)
    Map<String, String> columnMapping = new HashMap<>();
    columnMapping.put("CHARGING_INDICATOR", "CHARGING_INDICATOR");
    columnMapping.put("ORIG_AMOUNT_CURRENCY", "CURRENCY");
    columnMapping.put("FINAL_MOP", "FINAL_MOP");
    columnMapping.put("RECEIVER_BIC", "RECEIVER_BIC");
    columnMapping.put("PSD_INDICATOR", "PSD_INDICATOR");
    columnMapping.put("PAYMENT_DESTINATION_COUNTRY", "PYMT_DEST_CTRY");
    columnMapping.put("MESSAGE_TYPE", "SWIFT_MSG_TYP");
    columnMapping.put("DR_TRN_CODES", "DR-TRN_CODES");
    columnMapping.put("CR_TRN_CODES", "DR_TRN_CODES");
    columnMapping.put("FI_CHARGING_INDICATOR", "FI_CHARGING_INDICATOR");

    for (Map<String, String> outputRow : outputData) {
        String billingCode = outputRow.get("YOUR_REFERENCE_NO");
        List<Map<String, String>> nahRuleRows = nahRulesMap.get(billingCode);
        Map<String, String> resultRow = new HashMap<>(outputRow);

        // Initialize the comparison result
        resultRow.put("Comparison Result", "Failure");

        // Initialize lists to keep track of mismatched columns for both conditions
        List<String> mismatchedColumnsCondition1 = new ArrayList<>();
        List<String> mismatchedColumnsCondition2 = new ArrayList<>();

        boolean condition1Match = false;  // Initially set to false, we'll try to find a match
        boolean condition2Match = true;   // Assume condition 2 is true initially

        // Condition 1: Compare based on the columns relevant to billingCode
        if (nahRuleRows != null) {
            for (Map<String, String> nahRuleRow : nahRuleRows) {
                boolean currentCondition1Match = true;
                List<String> currentMismatchedColumns = new ArrayList<>();

                // Compare columns based on the defined mapping
                for (Map.Entry<String, String> entry : columnMapping.entrySet()) {
                    String outputColumn = entry.getKey();
                    String nahRulesColumn = entry.getValue();

                    String outputValue = outputRow.get(outputColumn);
                    String nahRuleValue = nahRuleRow.get(nahRulesColumn);

                    // Check if output value is present in the nah rules value
                    if (!isValuePresent(outputValue, nahRuleValue)) {
                        currentCondition1Match = false;
                        currentMismatchedColumns.add(outputColumn);  // Add to mismatched columns for condition 1
                    }
                }

                if (currentCondition1Match) {
                    condition1Match = true;  // We found a matching condition 1
                    break;  // No need to check further nahRuleRows once a match is found
                } else {
                    mismatchedColumnsCondition1.addAll(currentMismatchedColumns);
                }
            }
        }

        // Condition 2: Compare based on INITIAL_PRICE_ITEM_CD (same logic as previous)
        String initialPriceItemCd = outputRow.get("INITIAL_PRICE_ITEM_CD");
        Map<String, String> nahRulesForItemCd = nahRulesMap.get(initialPriceItemCd);

        if (nahRulesForItemCd != null) {
            for (Map.Entry<String, String> entry : columnMapping.entrySet()) {
                String outputColumn = entry.getKey();
                String nahRulesColumn = entry.getValue();

                String outputValue = outputRow.get(outputColumn);
                String nahRuleValue = nahRulesForItemCd.get(nahRulesColumn);

                // Check if output value is present in the nah rules value for this item code
                if (!isValuePresent(outputValue, nahRuleValue)) {
                    condition2Match = false;
                    mismatchedColumnsCondition2.add(outputColumn);  // Add to mismatched columns for condition 2
                }
            }
        }

        // Set the final comparison result
        if (condition1Match && condition2Match) {
            resultRow.put("Comparison Result", "Success");
        }

        // Add mismatched columns for both conditions
        if (!mismatchedColumnsCondition1.isEmpty()) {
            resultRow.put("Mismatched Columns Condition 1", String.join(", ", mismatchedColumnsCondition1));
        }
        if (!mismatchedColumnsCondition2.isEmpty()) {
            resultRow.put("Mismatched Columns Condition 2", String.join(", ", mismatchedColumnsCondition2));
        }

        results.add(resultRow);
    }
    return results;
}

// Helper method to check if value in output is present in nah rules value
private static boolean isValuePresent(String outputValue, String nahRuleValue) {
    if (outputValue == null || nahRuleValue == null) {
        return false;
    }

    // Split values by commas or spaces and check if output value is contained in nah rules value
    Set<String> nahRuleSet = new HashSet<>(Arrays.asList(nahRuleValue.split("[,\\s]+")));
    return nahRuleSet.contains(outputValue.trim());
}
```

### Key Changes:

1. **Multiple Entries for `BILLING_CODE`**:
   - `nahRulesMap` now holds a `List<Map<String, String>>` for each `BILLING_CODE` to account for multiple entries with different column values.
   
2. **Condition 1**:
   - The `compareWithNahRules` function iterates through all `nahRuleRows` for each `BILLING_CODE` and compares the corresponding columns. If any `nahRuleRow` matches the `outputRow`, it marks the condition as a match.

3. **Condition 2**:
   - The logic for comparing `INITIAL_PRICE_ITEM_CD` remains similar, and mismatched columns are tracked separately for both conditions.

This solution ensures that you correctly handle multiple occurrences of the same `BILLING_CODE` and compare all
