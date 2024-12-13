To search for `LOCAL_REFERENCE_NO` by its column name rather than relying on its index, you can first identify the index of the `LOCAL_REFERENCE_NO` column by searching for the header row. This is done by iterating through the first row (which typically contains column names) and matching the column name to find the index. Once the index is identified, you can use it to fetch the data from subsequent rows.

Here’s how you can modify the code to dynamically find the column index based on its name, rather than assuming it’s in a specific position.

### Modified Code to Search by Column Name

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.util.*;

public class ExcelProcessor {

    public static void main(String[] args) throws Exception {
        String file1Path = "path/to/first/excel/file.xlsx";  // Path to the first Excel file
        String file2Path = "path/to/second/excel/file.xlsx"; // Path to the second Excel file

        // Read data from both Excel files
        Map<String, Map<String, String>> resultMap = processExcelFiles(file1Path, file2Path);

        // Print the result map
        printResultMap(resultMap);
    }

    private static Map<String, Map<String, String>> processExcelFiles(String file1Path, String file2Path) throws Exception {
        // Load the first Excel file (contains RESULT and LOCAL_REFERENCE_NO columns)
        FileInputStream file1 = new FileInputStream(new File(file1Path));
        Workbook workbook1 = new XSSFWorkbook(file1);
        Sheet sheet1 = workbook1.getSheetAt(0);

        // Load the second Excel file (contains LOCAL_REFERENCE_NO and required data columns)
        FileInputStream file2 = new FileInputStream(new File(file2Path));
        Workbook workbook2 = new XSSFWorkbook(file2);
        Sheet sheet2 = workbook2.getSheetAt(0);

        Map<String, Map<String, String>> resultMap = new HashMap<>();

        // Read columns of interest from the second file into a map
        Map<String, List<String>> file2Data = new HashMap<>();
        int localReferenceNoIndexFile2 = getColumnIndex(sheet2, "LOCAL_REFERENCE_NO");  // Find index of LOCAL_REFERENCE_NO
        for (Row row : sheet2) {
            String localReferenceNo = getCellValue(row, localReferenceNoIndexFile2);  // Get value using dynamic index
            if (localReferenceNo != null) {
                List<String> rowData = new ArrayList<>();
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "CHARGING_INDICATOR")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "ORIG_AMOUNT_CURRENCY")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "FINAL_MOP")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "RECEIVER_BIC")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "PSD_INDICATOR")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "PAYMENT_DESTINATION_COUNTRY")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "MESSAGE_TYPE")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "DR_TRN_CODES")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "CR_TRN_CODES")));
                rowData.add(getCellValue(row, getColumnIndex(sheet2, "FI_CHARGING_INDICATOR")));
                file2Data.put(localReferenceNo, rowData);
            }
        }

        // Get index of the LOCAL_REFERENCE_NO and RESULT columns in the first file
        int localReferenceNoIndexFile1 = getColumnIndex(sheet1, "LOCAL_REFERENCE_NO");
        int resultIndexFile1 = getColumnIndex(sheet1, "RESULT");

        // Iterate over the first file and process the rows with "Incorrect Product Detected" in RESULT column
        for (Row row : sheet1) {
            String result = getCellValue(row, resultIndexFile1);  // Get value using dynamic index
            String localReferenceNo = getCellValue(row, localReferenceNoIndexFile1);  // Get value using dynamic index

            if ("Incorrect Product Detected".equals(result) && localReferenceNo != null) {
                String yourReference = getCellValue(row, getColumnIndex(sheet1, "YOUR_REFERENCE")); // Find YOUR_REFERENCE column

                // Retrieve corresponding data from the second file using LOCAL_REFERENCE_NO
                List<String> correspondingData = file2Data.get(localReferenceNo);
                if (correspondingData != null) {
                    Map<String, String> mapForYourReference = new HashMap<>();
                    mapForYourReference.put("CHARGING_INDICATOR", correspondingData.get(0));
                    mapForYourReference.put("ORIG_AMOUNT_CURRENCY", correspondingData.get(1));
                    mapForYourReference.put("FINAL_MOP", correspondingData.get(2));
                    mapForYourReference.put("RECEIVER_BIC", correspondingData.get(3));
                    mapForYourReference.put("PSD_INDICATOR", correspondingData.get(4));
                    mapForYourReference.put("PAYMENT_DESTINATION_COUNTRY", correspondingData.get(5));
                    mapForYourReference.put("MESSAGE_TYPE", correspondingData.get(6));
                    mapForYourReference.put("DR_TRN_CODES", correspondingData.get(7));
                    mapForYourReference.put("CR_TRN_CODES", correspondingData.get(8));
                    mapForYourReference.put("FI_CHARGING_INDICATOR", correspondingData.get(9));

                    // Put the data in the result map with YOUR_REFERENCE as the key
                    resultMap.put(yourReference, mapForYourReference);
                }
            }
        }

        workbook1.close();
        workbook2.close();

        return resultMap;
    }

    private static String getCellValue(Row row, int columnIndex) {
        Cell cell = row.getCell(columnIndex);
        if (cell != null) {
            switch (cell.getCellType()) {
                case STRING:
                    return cell.getStringCellValue();
                case NUMERIC:
                    return String.valueOf(cell.getNumericCellValue());
                default:
                    return null;
            }
        }
        return null;
    }

    private static int getColumnIndex(Sheet sheet, String columnName) {
        Row headerRow = sheet.getRow(0);  // Assuming the first row is the header row
        for (Cell cell : headerRow) {
            if (columnName.equals(cell.getStringCellValue())) {
                return cell.getColumnIndex();
            }
        }
        throw new IllegalArgumentException("Column " + columnName + " not found.");
    }

    private static void printResultMap(Map<String, Map<String, String>> resultMap) {
        // Print the map
        System.out.println("Result Map (YOUR_REFERENCE -> Data):");
        resultMap.forEach((yourReference, dataMap) -> {
            System.out.println("\nYOUR_REFERENCE: " + yourReference);
            dataMap.forEach((column, value) -> {
                System.out.println("\t" + column + ": " + value);
            });
        });
    }
}
```

### Explanation of Changes

1. **Dynamic Column Index Retrieval**:
   - **`getColumnIndex(Sheet sheet, String columnName)`**: This method searches for the column index by name. It assumes the first row of the sheet contains headers. The method iterates through the first row, and when it finds a column name matching the provided name, it returns the index of that column.
   
2. **Column Index Usage**:
   - For both Excel files (`file1` and `file2`), I used the `getColumnIndex` method to find the correct column index for `LOCAL_REFERENCE_NO`, `RESULT`, `YOUR_REFERENCE`, and other columns.
   - This allows the program to dynamically adapt if the column positions change or if the column names are different.

3. **Error Handling**:
   - If the specified column name is not found, the `getColumnIndex` method throws an `IllegalArgumentException`. You may want to handle this exception appropriately in a production scenario.

### Result Printing

The `printResultMap` method remains unchanged and will print the contents of the `resultMap`, with `YOUR_REFERENCE` as the key and its corresponding data for each row.

### Sample Output

The printed output remains the same as described before:

```
Result Map (YOUR_REFERENCE -> Data):

YOUR_REFERENCE: ref12345
    CHARGING_INDICATOR: Indicator1
    ORIG_AMOUNT_CURRENCY: USD
    FINAL_MOP: MOP1
    RECEIVER_BIC: BIC1
    PSD_INDICATOR: IndicatorA
    PAYMENT_DESTINATION_COUNTRY: US
    MESSAGE_TYPE: Type1
    DR_TRN_CODES: DR123
    CR_TRN_CODES: CR123
    FI_CHARGING_INDICATOR: FI123

YOUR_REFERENCE: ref67890
    CHARGING_INDICATOR: Indicator2
    ORIG_AMOUNT_CURRENCY: EUR
    FINAL_MOP: MOP2
    RECEIVER_BIC: BIC2
    PSD_INDICATOR: IndicatorB
    PAYMENT_DESTINATION_COUNTRY: UK
    MESSAGE_TYPE: Type2
    DR_TRN_CODES: DR456
    CR_TRN_CODES: CR456
    FI_CHARGING_INDICATOR: FI456
```

This ensures that the code works dynamically even if the column positions in the Excel files are modified.
