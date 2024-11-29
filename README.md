Here’s the updated code with **SLF4J logging** integrated. Logs are configured to display error messages on the terminal, along with appropriate log levels for better monitoring.

### Maven Dependency for SLF4J and Logging Framework

Add the following dependencies for SLF4J and a logging backend such as **Logback** in your `pom.xml`:

```xml
<dependencies>
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>

    <!-- Logback Classic for SLF4J implementation -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
</dependencies>
```

### Updated Java Code with SLF4J

```java
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.metadata.CompressionCodecName;
import org.apache.parquet.schema.MessageType;
import org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName;
import org.apache.parquet.schema.Types;
import org.apache.parquet.example.data.Group;
import org.apache.parquet.example.data.simple.SimpleGroup;
import org.apache.parquet.hadoop.example.GroupWriteSupport;
import org.apache.poi.ss.usermodel.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

public class ExcelToParquet {

    private static final Logger logger = LoggerFactory.getLogger(ExcelToParquet.class);

    public static void main(String[] args) {
        String excelFilePath = "input.xlsx";
        String parquetFilePath = "output.parquet";

        try (FileInputStream fileInputStream = new FileInputStream(new File(excelFilePath))) {
            logger.info("Reading Excel file: {}", excelFilePath);
            Workbook workbook = WorkbookFactory.create(fileInputStream);
            Sheet sheet = workbook.getSheetAt(0); // Read the first sheet
            
            logger.info("Generating Parquet schema...");
            MessageType schema = generateParquetSchema(sheet);

            logger.info("Writing Parquet file: {}", parquetFilePath);
            writeParquet(sheet, schema, parquetFilePath);
            
            logger.info("Conversion completed successfully. Parquet file saved at: {}", parquetFilePath);
        } catch (IOException e) {
            logger.error("Error processing the files: {}", e.getMessage(), e);
            System.err.println("Error: " + e.getMessage()); // Print to terminal
        }
    }

    private static MessageType generateParquetSchema(Sheet sheet) {
        Row headerRow = sheet.getRow(0); // Assume the first row as header
        Types.MessageTypeBuilder builder = Types.buildMessage();
        for (Cell cell : headerRow) {
            String columnName = cell.getStringCellValue();
            builder = builder.optional(PrimitiveTypeName.BINARY).named(columnName);
        }
        logger.info("Parquet schema generated with {} columns.", headerRow.getLastCellNum());
        return builder.named("ExcelSchema");
    }

    private static void writeParquet(Sheet sheet, MessageType schema, String parquetFilePath) throws IOException {
        GroupWriteSupport.setSchema(schema, new org.apache.hadoop.conf.Configuration());
        try (ParquetWriter<Group> writer = ExampleParquetWriter.builder(new org.apache.hadoop.fs.Path(parquetFilePath))
                .withType(schema)
                .withCompressionCodec(CompressionCodecName.SNAPPY)
                .withWriteMode(org.apache.parquet.hadoop.ParquetFileWriter.Mode.OVERWRITE)
                .build()) {

            for (int i = 1; i <= sheet.getLastRowNum(); i++) { // Start from the second row
                Row row = sheet.getRow(i);
                Group group = new SimpleGroup(schema);
                for (int j = 0; j < row.getLastCellNum(); j++) {
                    Cell cell = row.getCell(j);
                    if (cell != null) {
                        group.add(j, cell.toString());
                    } else {
                        group.add(j, ""); // Add empty string for null cells
                    }
                }
                writer.write(group);
            }
            logger.info("Parquet file writing completed.");
        }
    }
}
```

### Log Levels and Output Behavior

1. **INFO**: Provides status updates for each major step in the process, such as reading files, generating schemas, and writing Parquet files.
2. **ERROR**: Captures any exceptions and logs a detailed message with the exception stack trace.
3. **System.err**: Explicitly prints errors to the terminal for visibility.

### Logging Output Example

**On Success:**
```
INFO  [main] ExcelToParquet - Reading Excel file: input.xlsx
INFO  [main] ExcelToParquet - Generating Parquet schema...
INFO  [main] ExcelToParquet - Parquet schema generated with 3 columns.
INFO  [main] ExcelToParquet - Writing Parquet file: output.parquet
INFO  [main] ExcelToParquet - Parquet file writing completed.
INFO  [main] ExcelToParquet - Conversion completed successfully. Parquet file saved at: output.parquet
```

**On Error:**
```
ERROR [main] ExcelToParquet - Error processing the files: input.xlsx (No such file or directory)
java.io.FileNotFoundException: input.xlsx (No such file or directory)
Error: input.xlsx (No such file or directory)
```

### Advantages of SLF4J with Logback
- **Flexibility**: Allows switching logging frameworks if needed without code changes.
- **Granular Control**: Enables detailed configuration of log levels and outputs via `logback.xml`.
