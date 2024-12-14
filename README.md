private static List<Map<String, String>> compareWithNahRules(List<Map<String, String>> outputData, Map<String, Map<String, String>> nahRulesMap) {
    List<Map<String, String>> results = new ArrayList<>();

    // Define the relevant columns for comparison
    List<String> relevantColumns = Arrays.asList(
            "CHARGING_INDICATOR", "ORIG_AMOUNT_CURRENCY", "FINAL_MOP", "RECEIVER_BIC",
            "PSD_INDICATOR", "PAYMENT_DESTINATION_COUNTRY", "MESSAGE_TYPE",
            "DR_TRN_CODES", "CR_TRN_CODES", "FI_CHARGING_INDICATOR"
    );

    for (Map<String, String> outputRow : outputData) {
        String billingCode = outputRow.get("YOUR_REFERENCE_NO");
        Map<String, String> nahRuleRow = nahRulesMap.get(billingCode);
        Map<String, String> resultRow = new HashMap<>(outputRow);

        // Initialize the comparison result
        resultRow.put("Comparison Result", "Failure");

        // Initialize a list to keep track of mismatched columns
        List<String> mismatchedColumns = new ArrayList<>();

        if (nahRuleRow != null) {
            boolean allMatch = true;
            for (String key : relevantColumns) {
                String outputValue = outputRow.get(key);
                String nahRuleValue = nahRuleRow.get(key);

                if (!Objects.equals(outputValue, nahRuleValue)) {
                    allMatch = false;
                    mismatchedColumns.add(key); // Add the mismatched column to the list
                }
            }

            // If all relevant columns match, set the result to "Success"
            if (allMatch) {
                resultRow.put("Comparison Result", "Success");
            }
        }

        // Add the mismatched columns to the result row, if any
        if (!mismatchedColumns.isEmpty()) {
            resultRow.put("Mismatched Columns", String.join(", ", mismatchedColumns));
        }

        results.add(resultRow);
    }
    return results;
}
