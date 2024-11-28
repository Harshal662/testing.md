import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;

public class PipeSeparatedToExcel {

    public static void main(String[] args) {
        String inputFilePath = "data.dat"; // Path to your .dat file
        String outputFilePath = "output.xlsx"; // Path for the Excel file

        try {
            // Read all lines from the .dat file
            List<String> lines = Files.readAllLines(Paths.get(inputFilePath));

            // Remove the first and last lines
            if (lines.size() > 2) { // Ensure there are enough lines to process
                lines = lines.subList(1, lines.size() - 1);
            } else {
                System.err.println("The file has insufficient lines to process after removing first and last.");
                return;
            }

            // Create a new workbook and sheet
            Workbook workbook = new XSSFWorkbook();
            Sheet sheet = workbook.createSheet("Sheet1");

            // Process each line and split by '|'
            int rowIndex = 0;
            for (String line : lines) {
                Row row = sheet.createRow(rowIndex++);
                String[] values = line.split("\\|");
                for (int colIndex = 0; colIndex < values.length; colIndex++) {
                    Cell cell = row.createCell(colIndex);
                    cell.setCellValue(values[colIndex].trim());
                }
            }

            // Auto-size all columns
            for (int colIndex = 0; colIndex < sheet.getRow(0).getPhysicalNumberOfCells(); colIndex++) {
                sheet.autoSizeColumn(colIndex);
            }

            // Write the data to an Excel file
            try (FileOutputStream fileOut = new FileOutputStream(outputFilePath)) {
                workbook.write(fileOut);
            }

            workbook.close();
            System.out.println("Data successfully written to " + outputFilePath);

        } catch (IOException e) {
            System.err.println("Error processing file: " + e.getMessage());
        }
    }
}
