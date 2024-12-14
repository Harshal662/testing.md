To modify the `loadOutputData` method to extract only the specified columns instead of all columns, you can filter the `columnIndexMap` to retain only the desired column names. Here's the updated method:

```java
private static List<Map<String, String>> loadOutputData(Sheet sheet, List<String> filteredLocalRefNos) {
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
            outputData.add(rowData);
        }
    }
    return outputData;
}
```

### Key Changes:
1. **Filter Columns**: Added a `requiredColumns` list to specify the columns that should be extracted.
2. **Map Filtering**: The `columnIndexMap` now only includes indices of the required columns.
3. **Extraction**: During row processing, only the required columns are fetched and added to `rowData`.

This ensures that only the specified columns are included in the output, meeting your requirement.
