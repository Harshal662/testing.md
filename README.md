```
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
