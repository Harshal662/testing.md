private static List<Map<String, String>> loadOutputData(Sheet sheet, List<String> filteredLocalRefNos) {
    List<Map<String, String>> outputData = new ArrayList<>();
    Row headerRow = sheet.getRow(0);
    
    // Create a map of column names to column indices
    Map<String, Integer> columnIndexMap = new HashMap<>();
    for (int j = 0; j < headerRow.getLastCellNum(); j++) {
        String columnName = getCellValue(headerRow.getCell(j));
        columnIndexMap.put(columnName, j);
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
