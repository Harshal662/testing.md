
    private static void saveFilteredData(List<Map<String, String>> filteredData, String outputFilePath) throws IOException {
        try (Workbook workbook = new XSSFWorkbook()) {
            Sheet sheet = workbook.createSheet("Filtered Data");

            // Create header row
            Row headerRow = sheet.createRow(0);
            String[] headers = {"UDF_CHAR_9", "UDF_CHAR_28", "INITIAL_PRICE_ITEM_CD", "PRICEITEM_CD"};
            for (int i = 0; i < headers.length; i++) {
                headerRow.createCell(i).setCellValue(headers[i]);
            }

            // Fill data rows
            int rowIndex = 1;
            for (Map<String, String> row : filteredData) {
                Row dataRow = sheet.createRow(rowIndex++);
                dataRow.createCell(0).setCellValue(row.get("UDF_CHAR_9"));
                dataRow.createCell(1).setCellValue(row.get("UDF_CHAR_28"));
                dataRow.createCell(2).setCellValue(row.get("INITIAL_PRICE_ITEM_CD"));
                dataRow.createCell(3).setCellValue(row.get("PRICEITEM_CD"));
            }

            // Write to file
            try (FileOutputStream fos = new FileOutputStream(outputFilePath)) {
                workbook.write(fos);
            }
        }
    }

