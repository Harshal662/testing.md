import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.*;
import java.util.stream.Collectors;

public class ExcelProcessor {

    public static void main(String[] args) throws IOException {
        String resultsFilePath = "results.xlsx";
        String outputFilePath = "output.xlsx";
        String nahRulesFilePath = "NAH-rules.xlsx";
        String successFailureFilePath = "comparison_results.xlsx";

        // Load Excel files
        Workbook resultsWorkbook = new XSSFWorkbook(new FileInputStream(resultsFilePath));
        Workbook outputWorkbook = new XSSFWorkbook(new FileInputStream(outputFilePath));
        Workbook nahRulesWorkbook = new XSSFWorkbook(new FileInputStream(nahRulesFilePath));

        // Extract required sheets
        Sheet resultsSheet = resultsWorkbook.getSheetAt(0);
        Sheet outputSheet = outputWorkbook.getSheetAt(0);
        Sheet nahRulesSheet = nahRulesWorkbook.getSheetAt(0);

        // Get NAH-Rules data
        Map<String, Map<String, String>> nahRulesMap = loadNahRules(nahRulesSheet);

        // Filter "Incorrect Product Detected" rows from Results file
        List<String> filteredLocalRefNos = filterResults(resultsSheet);

        // Match and compare Output file rows based on filtered Local Reference Nos
        List<Map<String, String>> outputData = loadOutputData(outputSheet, filteredLocalRefNos);

        // Compare with NAH-Rules
        List<Map<String, String>> comparisonResults = compareWithNahRules(outputData, nahRulesMap);

        // Write results to Success/Failure file
        writeComparisonResults(comparisonResults, successFailureFilePath);

        // Close all workbooks
        resultsWorkbook.close();
        outputWorkbook.close();
        nahRulesWorkbook.close();
    }

    private static Map<String, Map<String, String>> loadNahRules(Sheet sheet) {
        Map<String, Map<String, String>> nahRulesMap = new HashMap<>();
        Row headerRow = sheet.getRow(0);
        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row == null) continue;
            
            String billingCode = getCellValue(row.getCell(0));
            Map<String, String> columnValues = new HashMap<>();
            for (int j = 1; j < row.getLastCellNum(); j++) {
                String header = getCellValue(headerRow.getCell(j));
                String value = getCellValue(row.getCell(j));
                columnValues.put(header, value);
            }
            nahRulesMap.put(billingCode, columnValues);
        }
        return nahRulesMap;
    }

    private static List<String> filterResults(Sheet sheet) {
        List<String> filteredLocalRefNos = new ArrayList<>();
        int localRefIndex = findColumnIndex(sheet.getRow(0), "LOCAL_REFERENCE_NO");
        int resultIndex = findColumnIndex(sheet.getRow(0), "RESULT");

        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row == null) continue;
            String result = getCellValue(row.getCell(resultIndex));
            if ("Incorrect Product Detected".equals(result)) {
                filteredLocalRefNos.add(getCellValue(row.getCell(localRefIndex)));
            }
        }
        return filteredLocalRefNos;
    }

    private static List<Map<String, String>> loadOutputData(Sheet sheet, List<String> filteredLocalRefNos) {
        List<Map<String, String>> outputData = new ArrayList<>();
        Row headerRow = sheet.getRow(0);
        
        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row == null) continue;
            String localRefNo = getCellValue(row.getCell(0));
            if (filteredLocalRefNos.contains(localRefNo)) {
                Map<String, String> rowData = new HashMap<>();
                for (int j = 0; j < row.getLastCellNum(); j++) {
                    String header = getCellValue(headerRow.getCell(j));
                    String value = getCellValue(row.getCell(j));
                    rowData.put(header, value);
                }
                outputData.add(rowData);
            }
        }
        return outputData;
    }

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
            resultRow.put("Comparison Result", "Failure");

            if (nahRuleRow != null) {
                boolean allMatch = true;
                for (String key : relevantColumns) {
                    if (!Objects.equals(outputRow.get(key), nahRuleRow.get(key))) {
                        allMatch = false;
                        break;
                    }
                }
                if (allMatch) {
                    resultRow.put("Comparison Result", "Success");
                }
            }
            results.add(resultRow);
        }
        return results;
    }

    private static void writeComparisonResults(List<Map<String, String>> results, String filePath) throws IOException {
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("Results");

        // Write header row
        if (!results.isEmpty()) {
            Row headerRow = sheet.createRow(0);
            int colIndex = 0;
            for (String key : results.get(0).keySet()) {
                headerRow.createCell(colIndex++).setCellValue(key);
            }

            // Write data rows
            int rowIndex = 1;
            for (Map<String, String> result : results) {
                Row row = sheet.createRow(rowIndex++);
                colIndex = 0;
                for (String key : result.keySet()) {
                    row.createCell(colIndex++).setCellValue(result.get(key));
                }
            }
        }

        try (FileOutputStream fos = new FileOutputStream(filePath)) {
            workbook.write(fos);
        }
        workbook.close();
    }

    private static int findColumnIndex(Row row, String columnName) {
        for (int i = 0; i < row.getLastCellNum(); i++) {
            if (row.getCell(i).getStringCellValue().equals(columnName)) {
                return i;
            }
        }
        throw new IllegalArgumentException("Column not found: " + columnName);
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
                    return String.valueOf((int) cell.getNumericCellValue());
                }
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case FORMULA:
                return cell.getCellFormula();
            default:
                return "";
        }
    }
}
