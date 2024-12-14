private static List<Map<String, String>> loadOutputData(Sheet sheet, List<String> filteredLocalRefNos, Sheet resultsSheet) {
    List<Map<String, String>> outputData = new ArrayList<>();
    Row headerRow = sheet.getRow(0);

    // Define the required column names
    List<String> requiredColumns = Arrays.asList(
            "CHARGING_INDICATOR", "ORIG_AMOUNT_CURRENCY", "FINAL_MOP", "RECEIVER_BIC",
            "PSD_INDICATOR", "PAYMENT_DESTINATION_COUNTRY", "MESSAGE_TYPE",
            "DR_TRN_CODES", "CR_TRN_CODES", "FI_CHARGING_INDICATOR", "LOCAL_REFERENCE_NO"
    );

    // Create a map of required column names to their indices
    Map<String, Integer> columnIndexMap = new HashMap<>();
    for (int j = 0; j < headerRow.getLastCellNum(); j++) {
        String columnName = getCellValue(headerRow.getCell(j));
        if (requiredColumns.contains(columnName)) {
            columnIndexMap.put(columnName, j);
        }
    }

    // Find the INITIAL_PRICE_ITEM column index in the resultsSheet
    int initialPriceItemIndex = findColumnIndex(resultsSheet.getRow(0), "INITIAL_PRICE_ITEM");

    for (int i = 1; i <= sheet.getLastRowNum(); i++) {
        Row row = sheet.getRow(i);
        if (row == null) continue;

        String localRefNo = getCellValue(row.getCell(columnIndexMap.get("LOCAL_REFERENCE_NO")));
        if (filteredLocalRefNos.contains(localRefNo)) {
            Map<String, String> rowData = new HashMap<>();
            for (Map.Entry<String, Integer> entry : columnIndexMap.entrySet()) {
                String columnName = entry.getKey();
                int colIndex = entry.getValue();
                String value = getCellValue(row.getCell(colIndex));
                rowData.put(columnName, value);
            }

            // Find the corresponding INITIAL_PRICE_ITEM value from the resultsSheet
            for (int j = 1; j <= resultsSheet.getLastRowNum(); j++) {
                Row resultRow = resultsSheet.getRow(j);
                if (resultRow == null) continue;

                String resultLocalRefNo = getCellValue(resultRow.getCell(findColumnIndex(resultRow, "LOCAL_REFERENCE_NO")));
                if (localRefNo.equals(resultLocalRefNo)) {
                    String initialPriceItem = getCellValue(resultRow.getCell(initialPriceItemIndex));
                    rowData.put("INITIAL_PRICE_ITEM", initialPriceItem); // Add INITIAL_PRICE_ITEM to the rowData
                    break; // No need to continue once we find the matching LOCAL_REFERENCE_NO
                }
            }

            outputData.add(rowData);
        }
    }
    return outputData;
}
