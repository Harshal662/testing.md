The error occurs because the `ExampleParquetWriter` class is not directly included in the `parquet-avro` library you imported. Instead, you need to write Parquet files using the `ParquetWriter` class with proper configurations. Below is the corrected code.

### Corrected Code

```java
import org.apache.parquet.hadoop.ParquetWriter;
import org.apache.parquet.hadoop.metadata.CompressionCodecName;
import org.apache.parquet.schema.MessageType;
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
            builder = builder.optional(org.apache.parquet.schema.PrimitiveType.PrimitiveTypeName.BINARY).named(columnName);
        }
        logger.info("Parquet schema generated with {} columns.", headerRow.getLastCellNum());
        return builder.named("ExcelSchema");
    }

    private static void writeParquet(Sheet sheet, MessageType schema, String parquetFilePath) throws IOException {
        GroupWriteSupport.setSchema(schema, new org.apache.hadoop.conf.Configuration());
        try (ParquetWriter<Group> writer = new ParquetWriter<>(
                new org.apache.hadoop.fs.Path(parquetFilePath),
                new GroupWriteSupport(),
                CompressionCodecName.SNAPPY,
                ParquetWriter.DEFAULT_BLOCK_SIZE,
                ParquetWriter.DEFAULT_PAGE_SIZE)) {

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

### Changes Made
1. **Replaced `ExampleParquetWriter`**:
   - Replaced it with a manual configuration of `ParquetWriter` using `GroupWriteSupport`.

2. **Schema Setup**:
   - Used `GroupWriteSupport.setSchema(schema, Configuration)` to set up the schema for ParquetWriter.

3. **ParquetWriter Initialization**:
   - Properly initialized `ParquetWriter` with parameters for compression, block size, and page size.

### Ensure Proper Maven Dependencies
Ensure your `pom.xml` includes these dependencies:

```xml
<dependencies>
    <!-- Apache POI -->
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>5.2.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi-ooxml</artifactId>
        <version>5.2.3</version>
    </dependency>

    <!-- Apache Parquet -->
    <dependency>
        <groupId>org.apache.parquet</groupId>
        <artifactId>parquet-avro</artifactId>
        <version>1.13.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>3.3.6</version>
    </dependency>

    <!-- SLF4J with Logback -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
</dependencies>
```

### Output
- Logs will be displayed on the terminal with appropriate log levels (`INFO`, `ERROR`).
- Errors will also print to `System.err` for visibility.
