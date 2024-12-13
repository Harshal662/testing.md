To print the map, you can use the `forEach` method to iterate over the map and print its contents in a readable format. The `resultMap` contains `YOUR_REFERENCE` as the key and another map with the column names as keys and their corresponding values as values. Here’s how you can modify the code to print the map:

### Code Update for Printing the Map

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
        for (Row row : sheet2) {
            String localReferenceNo = getCellValue(row, 0); // Assuming LOCAL_REFERENCE_NO is in the first column
            if (localReferenceNo != null) {
                List<String> rowData = new ArrayList<>();
                rowData.add(getCellValue(row, 1)); // CHARGING_INDICATOR
                rowData.add(getCellValue(row, 2)); // ORIG_AMOUNT_CURRENCY
                rowData.add(getCellValue(row, 3)); // FINAL_MOP
                rowData.add(getCellValue(row, 4)); // RECEIVER_BIC
                rowData.add(getCellValue(row, 5)); // PSD_INDICATOR
                rowData.add(getCellValue(row, 6)); // PAYMENT_DESTINATION_COUNTRY
                rowData.add(getCellValue(row, 7)); // MESSAGE_TYPE
                rowData.add(getCellValue(row, 8)); // DR_TRN_CODES
                rowData.add(getCellValue(row, 9)); // CR_TRN_CODES
                rowData.add(getCellValue(row, 10)); // FI_CHARGING_INDICATOR
                file2Data.put(localReferenceNo, rowData);
            }
        }

        // Iterate over the first file and process the rows with "Incorrect Product Detected" in RESULT column
        for (Row row : sheet1) {
            String result = getCellValue(row, 4);  // Assuming RESULT is in the 5th column (0-indexed)
            String localReferenceNo = getCellValue(row, 0);  // Assuming LOCAL_REFERENCE_NO is in the first column

            if ("Incorrect Product Detected".equals(result) && localReferenceNo != null) {
                String yourReference = getCellValue(row, 1); // Assuming YOUR_REFERENCE is in the second column

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

### Explanation of the Print Logic

1. **`printResultMap` Method**:
   - This method prints the contents of the `resultMap` in a readable format.
   - It iterates over the `resultMap`, which contains the `YOUR_REFERENCE` as the key and a nested map (with column names as keys and corresponding values) as the value.
   - The output will show `YOUR_REFERENCE` followed by the column names and their values for each entry in the map.

### Sample Output

If you have two files and the result is stored in the map, the printed output will look something like:

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

This way, you can easily inspect the contents of the map in the console.
