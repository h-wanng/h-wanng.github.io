+++
author = "H. Wang"
title = "Easyexcel自定义列宽样式策略"
date = "2024-09-19"
description = "自定义列宽样式策略解决easyexcel中生成Excel列宽过大或小问题"
tags = [
    "Java", "Easyexcel"
]
categories = [
    "Java"
]

image = ""
+++
在easyExcel中使用LongestMatchColumnWidthStyleStrategy做列宽适配：
```java
private Integer dataLength(List<WriteCellData<?>> cellDataList, Cell cell, Boolean isHead) {
    if (isHead) {
        return cell.getStringCellValue().getBytes().length;
    } else {
        WriteCellData<?> cellData = (WriteCellData)cellDataList.get(0);
        CellDataTypeEnum type = cellData.getType();
        if (type == null) {
            return -1;
        } else {
            switch (type) {
                case STRING:
                    return cellData.getStringValue().getBytes().length;
                case BOOLEAN:
                    return cellData.getBooleanValue().toString().getBytes().length;
                case NUMBER:
                    return cellData.getNumberValue().toString().getBytes().length;
                default:
                    return -1;
            }
        }
    }
}
```
由于此策略中获取宽度逻辑为直接直接byte长度，导致如果三字节的字符（中文）过多就会变得很宽，一字节（英文）的字符过多就会不够宽

重新编写自定义列宽策略实现根据字符的字节长度设置列宽: 
```java
public class CustomColumnWidthStyleStrategy extends AbstractColumnWidthStyleStrategy {
    private static final int MAX_COLUMN_WIDTH = 255;
    private final Map<Integer, Map<Integer, Integer>> cache = MapUtils.newHashMapWithExpectedSize(8);

    public CustomColumnWidthStyleStrategy() {}

    protected void setColumnWidth(WriteSheetHolder writeSheetHolder, List<WriteCellData<?>> cellDataList, Cell cell, Head head, Integer relativeRowIndex, Boolean isHead) {
        boolean needSetWidth = isHead || !CollectionUtils.isEmpty(cellDataList);
        if (needSetWidth) {
            Map<Integer, Integer> maxColumnWidthMap = this.cache.computeIfAbsent(writeSheetHolder.getSheetNo(), (key) -> new HashMap<>(16));
            double columnWidth = this.dataLength(cellDataList, cell, isHead);
            if (columnWidth >= 0) {
                if (columnWidth > MAX_COLUMN_WIDTH) {
                    columnWidth = MAX_COLUMN_WIDTH;
                }

                Integer maxColumnWidth = maxColumnWidthMap.get(cell.getColumnIndex());
                if (maxColumnWidth == null || columnWidth > maxColumnWidth) {
                    maxColumnWidthMap.put(cell.getColumnIndex(), (int) columnWidth);
                    writeSheetHolder.getSheet().setColumnWidth(cell.getColumnIndex(), (int) columnWidth * 256);
                }

            }
        }
    }

    private double dataLength(List<WriteCellData<?>> cellDataList, Cell cell, Boolean isHead) {
        if (isHead) {
            return getExcelWidth(cell.getStringCellValue());
        } else {
            WriteCellData<?> cellData = cellDataList.get(0);
            CellDataTypeEnum type = cellData.getType();
            if (type == null) {
                return 0;
            } else {
                switch (type) {
                    case STRING:
                        return this.getExcelWidth(cellData.getStringValue());
                    case BOOLEAN:
                        return this.getExcelWidth(cellData.getBooleanValue().toString());
                    case NUMBER:
                        return this.getExcelWidth(cellData.getNumberValue().toString());
                    default:
                        return 0;
                }
            }
        }
    }

    /**
     * 调整单元格字符字节宽度
     */
    private double getExcelWidth(String str) {
        double length = 0;
        char[] chars = str.toCharArray();
        for (char c : chars) {
            byte[] bytes = this.getUtf8Bytes(c);
            if (bytes.length == 1) {
                length += 1.6;
            }
            if (bytes.length == 2) {
                length += 2.2;
            }
            if (bytes.length == 3) {
                length += 3.85;
            }
            if (bytes.length == 4) {
                length += 4.2;
            }
        }
        return length;
    }

    private byte[] getUtf8Bytes(char c) {
        char[] chars = {c};
        CharBuffer charBuffer = CharBuffer.allocate(chars.length);
        charBuffer.put(chars);
        charBuffer.flip();
        ByteBuffer byteBuffer = StandardCharsets.UTF_8.encode(charBuffer);
        return byteBuffer.array();
    }
}
```
