# pointer to pointer in C

在实现矩阵标准乘法时，通过从数据文本中读取矩阵元素至二维数组来建立矩阵。设计buildMatrix函数来实现该功能，函数最初的声明如下：<br> 
 ```C
 /*
 fp:      指向数据文件的文件指针
 matrix:  指向矩阵二维数组的指针
 pRow:    指向矩阵行数的指针
 pCol：   指向矩阵列数的指针
 */
 bool buildMatrix(FILE *fp, int **matrix, int *pRow, int *pCol);
```
 这里传入buildMatrix的四个参数，只有文件指针fp是buildMatrix所需要的，而另外的三个参数则是用于给caller返回矩阵相关信息。
 buildMatrix函数实现如下:
 ```C
 bool buildMatrix(FILE *fp, int **matrix, int *pRow, int *pCol)
{
    int row, col;
    fscanf(fp, "%d%d", pRow, pCol);
    
    row = *pRow;
    col = *pCol;
    if (row < 1 && col < 1)
    {
        //invalid data!
        return false;
    }
    
    matrix = malloc(sizeof(int*) * row);
    if(!matrix)
    {
        return false;
    }
    for(int i = 0; i < row; i++)
    {
        matrix[i] = malloc(sizeof(int) * col);
        for(int j = 0; j < col; j++)
        {
            fscanf(fp, "%d", &matrix[i][j]); 
        }
    }   
    return true;
}
 ```

以下是调用函数的代码段:
```C
    int **matrix1, row1, col1;
    int **matrix2, row2, col2;
    int **matrix3, row3, col3;
    FILE *fp = NULL;
    
    char *filename = "./data.txt";
    if(!(fp = fopen(filename, "r")))
    {
        printf("Openning file '%s' failed!\n", filename);
        return -1;
    }
    
    //error1
    if(!buildMatrix(fp, matrix1, &row1, &col1) ||
       !buildMatrix(fp, matrix2, &row2, &col2))
       {
            //...
       }
 ```
这是一段有bug的代码，试图给buildMatrix传入参数matrix，然后将函数内部动态申请的二维数组的首地址赋值给matrix。
但问题有两点:  
>>1. matrix1和matrix2都仅仅是声明的指针变量，并没有初始化。作为实参传入函数，没有任何意义！ 
>>2. buildMatrix函数的第二个参数实际上是用于返回函数内部动态申请的二维数组的首地址。

为了能够通过函数参数给caller返回函数内部动态申请的二维数组首地址，先来分析一下如何通过函数参数给caller函数返回一个整数。
可以参考buildMatrix的后两个参数，这两个参数用于给caller返回矩阵的行数和列数，而形参的类型是int*。
以此类推，为了给caller返回一个二维数组的首地址，形参的类型就应该是int***。<br>

所以buildMatrix的声明应改为：
```C
bool buildMatrix(FILE *fp, int ***pMatrix, int *pRow, int *pCol);
```
而caller中的代码段应改为:
```C
    if(!buildMatrix(fp, &matrix1, &row1, &col1) ||
       !buildMatrix(fp, &matrix2, &row2, &col2))
       {
            //...
       }
```
