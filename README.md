To address your updated requirement, the code will now:

1. Match `LOCAL_REFERENCE_NO` from `output1.xlsx` with `UDF_CHAR_9` from `positive.xlsx`.
2. Include all rows from `output1.xlsx` in the output.
   - If a match is found, include the corresponding data from `positive.xlsx`.
   - If no match is found, leave the columns from `positive.xlsx` blank.

Here’s the updated code:

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.*;

public class ProcessExcelFiles {

    public static void main(String[] args) {
        String file1Path = "output1.xlsx"; // Replace with the path to output1.xlsx
        String file2Path = "positive.xlsx"; // Replace with the path to positive.xlsx
        String outputFilePath = "merged_data.xlsx"; // Output file path

        try {
            // Read specific columns from both files
            Map<String, String> file1Data = readFile1(file1Path);
            Map<String, Map<String, String>> file2Data = readFile2(file2Path);

            // Merge data
            List<Map<String, String>> mergedData = mergeData(file1Data, file2Data);

            // Save merged data to a new Excel file
            saveMergedData(mergedData, outputFilePath);

            System.out.println("Merged data has been saved to: " + outputFilePath);
        } catch (Exception e) {
            System.err.println("Error processing files: " + e.getMessage());
        }
    }

    private static Map<String, String> readFile1(String filePath) throws IOException {
        Map<String, String> data = new LinkedHashMap<>();

        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0);

            int localRefNoIndex = -1;
            int yourRefIndex = -1;

            Row headerRow = sheet.getRow(0);
            for (Cell cell : headerRow) {
                String header = cell.getStringCellValue();
                if ("LOCAL_REFERENCE_NO".equalsIgnoreCase(header)) {
                    localRefNoIndex = cell.getColumnIndex();
                } else if ("YOUR_REFERENCE".equalsIgnoreCase(header)) {
                    yourRefIndex = cell.getColumnIndex();
                }
            }

            if (localRefNoIndex == -1 || yourRefIndex == -1) {
                throw new IllegalArgumentException("Required columns not found in " + filePath);
            }

            for (Row row : sheet) {
                if (row.getRowNum() == 0) continue; // Skip header
                String localRefNo = getCellValue(row.getCell(localRefNoIndex));
                String yourRef = getCellValue(row.getCell(yourRefIndex));
                data.put(localRefNo, yourRef);
            }
        }
        return data;
    }

    private static Map<String, Map<String, String>> readFile2(String filePath) throws IOException {
        Map<String, Map<String, String>> data = new HashMap<>();

        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0);

            int udfChar9Index = -1;
            int udfChar28Index = -1;
            int initialPriceItemCdIndex = -1;
            int priceItemCdIndex = -1;

            Row headerRow = sheet.getRow(0);
            for (Cell cell : headerRow) {
                String header = cell.getStringCellValue();
                if ("UDF_CHAR_9".equalsIgnoreCase(header)) {
                    udfChar9Index = cell.getColumnIndex();
                } else if ("UDF_CHAR_28".equalsIgnoreCase(header)) {
                    udfChar28Index = cell.getColumnIndex();
                } else if ("INITIAL_PRICE_ITEM_CD".equalsIgnoreCase(header)) {
                    initialPriceItemCdIndex = cell.getColumnIndex();
                } else if ("PRICEITEM_CD".equalsIgnoreCase(header)) {
                    priceItemCdIndex = cell.getColumnIndex();
                }
            }

            if (udfChar9Index == -1 || udfChar28Index == -1 || initialPriceItemCdIndex == -1 || priceItemCdIndex == -1) {
                throw new IllegalArgumentException("Required columns not found in " + filePath);
            }

            for (Row row : sheet) {
                if (row.getRowNum() == 0) continue; // Skip header
                Map<String, String> rowData = new HashMap<>();
                String udfChar9 = getCellValue(row.getCell(udfChar9Index));
                rowData.put("UDF_CHAR_28", getCellValue(row.getCell(udfChar28Index)));
                rowData.put("INITIAL_PRICE_ITEM_CD", getCellValue(row.getCell(initialPriceItemCdIndex)));
                rowData.put("PRICEITEM_CD", getCellValue(row.getCell(priceItemCdIndex)));
                data.put(udfChar9, rowData);
            }
        }
        return data;
    }

    private static List<Map<String, String>> mergeData(Map<String, String> file1Data, Map<String, Map<String, String>> file2Data) {
        List<Map<String, String>> mergedData = new ArrayList<>();

        for (Map.Entry<String, String> entry : file1Data.entrySet()) {
            String localRefNo = entry.getKey();
            String yourRef = entry.getValue();

            Map<String, String> rowData = new LinkedHashMap<>();
            rowData.put("LOCAL_REFERENCE_NO", localRefNo);
            rowData.put("YOUR_REFERENCE", yourRef);

            Map<String, String> file2Row = file2Data.get(localRefNo);
            if (file2Row != null) {
                rowData.putAll(file2Row);
            } else {
                rowData.put("UDF_CHAR_28", "");
                rowData.put("INITIAL_PRICE_ITEM_CD", "");
                rowData.put("PRICEITEM_CD", "");
            }

            mergedData.add(rowData);
        }

        return mergedData;
    }

    private static void saveMergedData(List<Map<String, String>> mergedData, String outputFilePath) throws IOException {
        try (Workbook workbook = new XSSFWorkbook()) {
            Sheet sheet = workbook.createSheet("Merged Data");

            // Create header row
            Row headerRow = sheet.createRow(0);
            String[] headers = {"LOCAL_REFERENCE_NO", "YOUR_REFERENCE", "UDF_CHAR_28", "INITIAL_PRICE_ITEM_CD", "PRICEITEM_CD"};
            for (int i = 0; i < headers.length; i++) {
                headerRow.createCell(i).setCellValue(headers[i]);
            }

            // Fill data rows
            int rowIndex = 1;
            for (Map<String, String> row : mergedData) {
                Row dataRow = sheet.createRow(rowIndex++);
                dataRow.createCell(0).setCellValue(row.get("LOCAL_REFERENCE_NO"));
                dataRow.createCell(1).setCellValue(row.get("YOUR_REFERENCE"));
                dataRow.createCell(2).setCellValue(row.get("UDF_CHAR_28"));
                dataRow.createCell(3).setCellValue(row.get("INITIAL_PRICE_ITEM_CD"));
                dataRow.createCell(4).setCellValue(row.get("PRICEITEM_CD"));
            }

            // Write to file
            try (FileOutputStream fos = new FileOutputStream(outputFilePath)) {
                workbook.write(fos);
            }
        }
    }

    private static String getCellValue(Cell cell) {
        if (cell == null) return "";
        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue();
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    return cell.getDateCellValue().toString();
                } else {
                    return String.valueOf(cell.getNumericCellValue());
                }
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case FORMULA:
                return cell.getCellFormula();
            case BLANK:
                return "";
            default:
                return "UNKNOWN";
        }
    }
}
```

### Key Changes:
1. **`mergeData` Method**:
   - Iterates through all `LOCAL_REFERENCE_NO` in `output1.xlsx`.
   - Adds matching rows from `positive.xlsx` or leaves columns blank if no match is found.

2. **Output Format**:
   - `LOCAL_REFERENCE_NO` and `YOUR_REFERENCE` from `output1.xlsx` are always present.
   - Columns `UDF_CHAR_28`, `INITIAL_PRICE_ITEM_CD`, and `PRICEITEM_CD` are blank for unmatched rows.

3. **Saved File**:
   - The merged data is saved to `merged_data.xlsx`.
