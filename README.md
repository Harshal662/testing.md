import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroupFactory;
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.example.ExampleParquetWriter;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.Types;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;

public class ConvertExcelToParquet {

    public static void main(String[] args) {
        String excelDirectoryPath = "excel_files"; // Directory containing Excel files
        String outputDirectoryPath = "output_parquet"; // Directory to store Parquet files

        try {
            // Ensure the output directory exists
            Files.createDirectories(Paths.get(outputDirectoryPath));

            // Get all .xlsx files from the input directory
            List<File> excelFiles = getExcelFiles(excelDirectoryPath);

            if (excelFiles.isEmpty()) {
                System.out.println("No Excel files found in the directory.");
                return;
            }

            // Convert each Excel file to Parquet
            for (File excelFile : excelFiles) {
                System.out.println("Processing file: " + excelFile.getName());
                String parquetFileName = excelFile.getName().replace(".xlsx", ".parquet");
                String outputFilePath = Paths.get(outputDirectoryPath, parquetFileName).toString();
                convertToParquet(excelFile, outputFilePath);
                System.out.println("Converted to Parquet: " + outputFilePath);
            }
        } catch (Exception e) {
            System.err.println("Error during conversion: " + e.getMessage());
        }
    }

    // Method to get all .xlsx files from a directory
    private static List<File> getExcelFiles(String directoryPath) throws IOException {
        return Files.walk(Paths.get(directoryPath))
                .filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".xlsx"))
                .map(Path::toFile)
                .toList();
    }

    // Method to convert an Excel file to Parquet
    private static void convertToParquet(File excelFile, String outputFilePath) throws IOException, InvalidFormatException {
        try (FileInputStream fis = new FileInputStream(excelFile);
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0); // Convert only the first sheet

            // Generate the schema dynamically based on the number of columns in the Excel sheet
            MessageType schema = generateSchema(sheet.getRow(0)); // Get the schema from the first row (header)
            SimpleGroupFactory groupFactory = new SimpleGroupFactory(schema);

            try (ParquetWriter<Group> writer = ExampleParquetWriter.builder(new org.apache.hadoop.fs.Path(outputFilePath))
                    .withSchema(schema)
                    .build()) {

                for (Row row : sheet) {
                    if (row.getRowNum() == 0) continue; // Skip the header row
                    Group group = groupFactory.newGroup();

                    // Iterate over all columns in the row
                    for (int i = 0; i < row.getLastCellNum(); i++) {
                        Cell cell = row.getCell(i);
                        String value = (cell == null) ? "" : getCellValue(cell);
                        group.add("Column" + (i + 1), value); // Add value to the group
                    }

                    writer.write(group);
                }
            }

        } catch (Exception e) {
            System.err.println("Error converting " + excelFile.getName() + ": " + e.getMessage());
        }
    }

    // Method to generate the Parquet schema based on the number of columns in the first row (header)
    private static MessageType generateSchema(Row headerRow) {
        Types.MessageTypeBuilder schemaBuilder = Types.buildMessage();

        // Loop through the columns and create a field for each one in the schema
        for (int i = 0; i < headerRow.getLastCellNum(); i++) {
            // For this example, all columns are treated as OPTIONAL strings (BINARY)
            schemaBuilder.optional(Types.primitive(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY, Types.Repetition.OPTIONAL)
                    .as(org.apache.parquet.schema.OriginalType.UTF8)
                    .named("Column" + (i + 1))); // Naming columns as "Column1", "Column2", etc.
        }

        return schemaBuilder.named("ExcelData");
    }

    // Utility method to get cell value as a string
    private static String getCellValue(Cell cell) {
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
