## EasyExcel
alibaba的读写excel的框架，将文件加载到磁盘，使用按行读取解析的逻辑优化了因为文件过大导致的内存溢出问题。
### 实例  
```java
@Data
public class DemoExcelTest {
    @ExcelProperty("字符串标题")
    private String string;
    @ExcelProperty("日期标题")
    private Date date;
    @ExcelProperty("数字标题")
    private Double doubleDate;
    @ExcelIgnore
    private String ignore;
}

public class DemoExcelListener extends AnalysisEventListener<DemoExcelTest> {

    public DemoExcelListener(){

    }

    private static final int BACH_COUNT = 5;
    private List<DemoExcelTest> list = new ArrayList<>();
    private DemoExcelTest demoExcelTest;
    public DemoExcelListener(DemoExcelTest demoExcelTest){
        this.demoExcelTest = demoExcelTest;
    }

    /**
     * 按行读取
     * @param demoExcelTest
     * @param analysisContext
     */
    @Override
    public void invoke(DemoExcelTest demoExcelTest, AnalysisContext analysisContext) {
        System.out.println("读取了一条数据");
        list.add(demoExcelTest);
        if (list.size() > BACH_COUNT){
            for (DemoExcelTest excelTest : list) {
                System.out.println(excelTest.getString() + excelTest.getDate() + excelTest.getDoubleDate());
            }
            list.clear();
        }

    }

    /**
     * 文件读取完成后的动作
     * @param analysisContext
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {

    }
}

public class Test {
    /**
     * 将数据写入excel
     */
    @org.junit.Test
    public void simpleTest(){
        String fileName = "C:\\Users\\MrGao\\Desktop\\" + "simpleTest" + System.currentTimeMillis() + ".xlsx";
        //写入的文件名、格式、数据
        EasyExcel.write(fileName,DemoExcelTest.class).sheet("模板").doWrite(data());
    }

    @org.junit.Test
    public void simpleReaderTest(){
        String fileName = "C:\\Users\\MrGao\\Desktop\\simpleTest1676383365351.xlsx";
        //读取的文件路径、读取的格式、文件读取监听器
        EasyExcel.read(fileName,DemoExcelTest.class, new DemoExcelListener()).sheet().doRead();
    }

    public List<DemoExcelTest> data(){
        List<DemoExcelTest> list = new ArrayList<>();
        for (int i = 0;i < 10;i++){
            DemoExcelTest demoExcelTest = new DemoExcelTest();
            demoExcelTest.setString("字符串" + i);
            demoExcelTest.setDate(new Date());
            demoExcelTest.setDoubleDate(0.56);
            list.add(demoExcelTest);

        }
        return list;
    }
}
```




















    