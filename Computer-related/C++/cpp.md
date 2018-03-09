##段错误原因
####1.往受到系统保护的内存地址写数据
如:   
> int i=0;
   scanf("%d",i);//应该是&i

 再如:
 > 
int main(){
   char *p;  
   p = NULL;  
   *p = 'x';   //往内存地址0处写东西
   printf("%c", *p);  
   return 0;
}

又如:
> char** info;   //并未开辟可使用的空间
  for(int i = 0; i<=num-1;i++)
  {
    //char* info;
    info[i] = (char*)malloc(sizeof(char)*STRING_LENGTH);
    scanf("%s",info);   //注意最后一个参数, 应该是地址, info本身就是字符串的首地址了,不需要再加&符号
  }

正确的应该为:
>char** info = (char**)malloc(sizeof(char*)*num);   //开辟一段长度为num的空间,每个空间中存储一个地址,指向一个字符串
  for(int i = 0; i<=num-1;i++)
  {
    info[i] = (char*)malloc(sizeof(char)*STRING_LENGTH);  //以每个地址为起始地址,开辟一段长度为STRING_LENGTH的空间, 每个空间存储一个字符
    //  scanf("%s",info);  
  }

另外:
> typedef void (*VISIT)(int);
void visit(int a)
{
  printf("visit %d,",a);
}
int main()
{
  VISIT fn;  //声明一个VISIT类型的函数指针fn, 它随机指向内存中的一个位置（这个位置可能是别人已经在使用，或者不让他人使用的位置）
  //fn = visit;   //如果我们没有给它指向一个合理的位置, 在使用这个指针时就会引发段错误
  fn(3);

}

 可以通过man 7 signal | grep SEGV查看SIGSEGV的信息

####2.内存越界(数组越界\变量类型不一致等)
如:
	int a[5];  //这样定义的数组a中元素值未可知,并非全是0
	printf("%d", a[10]);  //超过了数组a的索引范围

 再如:
	char c='c';
	printf("%s", c); //试图把char型按照字符串格式输出，这里的**字符会解释成整数**，
             //再解释成地址，如果这个整数代表的地址不存在或者不可访问，自然也是访问了不该访问的


####如何避免出现段错误
 -  定义了指针后记得初始化, 在往地址写/读数据时, 记得判断指针是否为NULL
 -  在使用数组的时候记得**初始化!!**, 注意防止下标访问越界
 -  在处理变量的时候注意变量的格式是否合理

####如何发现程序中的段错误？
-  用gdb来调试，在运行到段错误的地方，会自动停下来并显示出错的行和行号.记得在**编译**的时候加上**-g**参数，用来显示调试信息
-  catchsegv命令
-  条件编译指令#ifdef DEBUG和#endif


##"->"与"."
对于下面的例子:
    typedef struct OtherNode
    { 
      int index;
      struct OtherNode* next;
    }OtherNode;

    typedef struct HeadNode
    {
      ElemType data;
      OtherNode* otnode;
    }HeadNode;

    typedef struct AdjListGraph
    {
      HeadNode headvec[MAX_NODE_NUM];
      int vexnum, arcnum;
      GKind kind;
    }AdjListGraph;


如果调用:
    int pos = G->headvec[i].otnode.index;

就会出现下面的错误提示:
    request for member ‘index’ in ‘G->AdjListGraph::headvec[i].HeadNode::otnode’, which is of pointer type ‘OtherNode*’ (maybe you meant to use ‘->’ ?)
应该改为:
    int pos = G->headvec[i].otnode->index;
    因为otnode是个指针类型, 指针类型的结构体访问成员的时候需要用"->"而不是"."