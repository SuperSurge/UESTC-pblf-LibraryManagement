# LibraryController接口文档

github仓库：https://github.com/Soulmate404/LibraryController.git

为了尽可能的搞的分数高一点，我做了大概的计划，涉及用户管理，权限管理，借记管理等等，接下来分块介绍

## 功能介绍

1. 用户登陆

   初始界面要求用户输入ID与密码来进行登陆并校验是否正确，同时记录用户的权限等级

2. 权限分级

   普通用户仅能进行查询与借书还书的操作，管理员可进行添加、删除书籍，添加、删除用户，查看全部用户，修改用户密码等操作

3. 图书管理

   数据库中记录图书的ID，书名，作者，剩余数量，位置，用于供读者查询，读者也可以根据ID借书还书

4. 管理员操作

## Mapper层接口介绍

1. 用户登陆

   ```c
   int CheckUser（char* id,char* pass_wd）
   ```

   调用以检查用户是否存在，返回具体权限等级（0为管理员，1为普通用户，-1为用户不存在）.

   ```c
   int ResetPass(int id，char* pass)
   ```

   修改密码（0为成功，-1为失败）

2. 图书管理

   ```c
   int CheckBook(int id)
   ```

   根据ID检查书籍是否存在，存在返回0，不存在返回-1

   ```c
   int GetBookNum(int id)
   ```

   返回书籍的剩余数量

   ```(
   MYSQL_ROW SelectByID(int id)
   ```

   通过ID检索书籍，直接理解为返回一个char*类型的数组即可，如果不存在则返回NULL

   ```c
   MYSQL_ROWS SelectByName(char* name)
   ```

   通过书名查找书籍，支持简单的模糊查询，返回`MYSQL_ROWS`，这是一个

   ```c
   int AddBorrow(int userid,char* name,int bookid,char* time)
   ```

   借书登记，输入借书人与对应书籍的ID,借书时间，返回0表示成功，-1表示失败

   ```c
   int DeleteBorrow(int userid,int bookid)
   ```

   还书登记，同样的，返回0表示成功，-1表示失败.

   ```c
   int CheckBorrow(int userid,int bookid)
   ```

   用以确认有对应的记录，有返回对应的数量，没有返回-1

3. 管理员操作

   ```c
   int CheckBook(int id)
   ```

   检查书籍是否存在

   ```c
   int CheckUser(int id)
   ```

   检查用户是否存在

   ```
   MYSQL_ROW SelectUser(int id)
   ```

   通过ID获取用户信息

   ```c
   int AddBooks(int id,char* name,char* writer,int last_num,char* position)
   ```

   增加书籍，`last_num`指剩余数量,返回值含义同上

   ```c
   int AddUser(int id,char* name,char* pass_wd,int authority)
   ```

   增加用户，`authority`指权限等级

   ```c
   int DeleteBook(int id)
   ```

   根据输入的id删除书籍

   ```c
   int DeleteUser(int id)
   ```

   根据ID删除用户

   ```c
   int UpdateUserAuthority(int id,int authority)
   ```

   修改用户权限等级

   ```
   MYSQL_ROWS CheckUserBorrow（int id）
   ```

   通过ID返回对应用户的借书情况，注意，由于res指针包含其他信息，即使不存在对应信息也不回返回NULL，需要额外的校验

   ```c
   MYSQL_ROWS CheckAllBorrow（）
   ```

   检索全部的借阅

   ```c
   MYSQL_RES* CheckBooksBoorow(int id)
   ```

   通过ID返回对应书籍的借阅情况

   ```c
   int RootResetPass(int id,char* pass)
   ```

   以管理员权限重置密码



## Service层接口介绍

Service层应当调用mapper层提供的接口对Controller层输入的数据进行处理与校验，并向Controller层返回结果是否成功，且应当注意对资源的释放,

1. 用户登陆

   ```c
   int login(int id,char* pass_wd)
   ```

   用户登陆，在Service层就要求调用mapper层的函数进行登陆，返回值应当为用户的权限值，注意先检查用户是否存在，若不存在则返回-1

   ```c
   char* resetPass(int id，char* c1,char* c2)
   ```

   向用户返回新修改后的密码,c1与c2是两次输入的密码，确保一致，不一致返回NULL

2. 图书借阅

   ```c
   int selectByID(int id)
   ```

   需要对mapper层返回的指针进行处理，完成对应数据的打印，若错误返回-1

   ```c
   int selectByName(char* name)
   ```

   通过书名查找书籍，支持简单的模糊查询，同样的返回`MYSQL_RES`指针，进行处理完成数据打印，返回值同上

   ```c
   int addBoorow(int userid,char* name,int bookid，char* time)
   ```

   借书登记，注意在登记前检查书籍是否存在以及书籍是否有剩余(0成功，1书籍不存在，2书籍无剩余)

   ```c
   int deleteBorrow(int userid,int bookid)
   ```

   还书登记，同样进行必要的检查，确认有此条记录再进行登记。

3. 后台管理

   在后台管理中应当确认在每一步确认输入的数据是有效的，具体的校验函数在mapper层中已全部完成，直接调用即可

   ```c
   int addBooks(int id,char* name,char* writer,int last_num,char* position)
   ```

   检查书籍后再进行添加，若出现重复问题则返回-1，成功返回0

   ```c
   int deletBooks(int id)
   ```

   同上

   ```c
   int addUser(int id,char* name,char* pass_wd,int authority)
   ```

   检查后添加用户

   ```c
   int deleteUser(int id)
   ```

   同上

   ```c
   int slelectUser(int id)
   ```

   查询用户信息，对mapper返回数据打印，返回值同上

   ```c
   int updateAuthority(int id,int authority)
   ```

   校验ID是否存在，authority是否为0或1以外的数，成功为0，ID错误为1，authority错误为2

   ```c
   int checkUserBorrow（int id）
   ```

   存在则打印，返回0，不存在则返回-1，若ID值为0，则返回全部记录

   ```c
   int checkBooksBorrow(int id)
   ```

   通过书籍ID查询借阅,处理同上

   ```c
   char* RootResetPass(int id,char* c1,char* c2)
   ```

   管理员权限重置密码，错误则返回NULL，正确返回对应的密码

Service层的接口就这么多，但要完成每一步的逻辑，一定要注意数据正确性的校验，并保证返回值的规范

另外，我这里贴一个处理`MYSQL_RES`指针并答应的方式

```c
MYSQL_RES *res = getResult(handle);//获取对应的指针
    if(res == NULL) {
        printf("error");
        return -2;
    }
    //判断结果行和列
    int rows = mysql_num_rows(res); //获取表的行数,这个函数与下面那些长得类似的都是来自于<mysql.h>
    printf("rows : %d \n",rows);
    int fields = mysql_num_fields(res); //获取表的列数
    printf("fields: %d\n",fields);

    MYSQL_ROW row;创建一个指向数组的指针，这个数组中的每个元素都是一个char* ,指向返回的数据行中的一个单独的数据
    while ((row = mysql_fetch_row(res))){//这里的判断是判断row是不是一个空指针,如果是则停止输出
        int i = 0;
        for (i = 0;  i< fields; i++) {
            printf("%s\t",row[i]);
        }
        printf("\n");
    }

   

    mysql_free_result(res);//在执行完毕后释放资源

```

## Controller层接口介绍

严格来讲这一部分不是纯粹的Controller层,还涉及到用户的互动,但毕竟没有进行前后端分离,只能这样了

这一层的函数我不会具体设计,只会提出要求,这些要求的底层实现已经完成,你只需要完成展示即可

1. 交互界面的设计

   具体要求如图

   ![屏幕截图 2024 11 29 143518](https://s1.imagehub.cc/images/2024/11/29/de94fd54a3c6e0363cdb56284a2fef0c.png)

   ![屏幕截图 2024 11 29 143733](https://s1.imagehub.cc/images/2024/11/29/7100f03722272702897c6b17bb6060a8.png)

当然不是完全的相同,需要根据Service的内容重写,这些本质上都是用printf和System系列的函数做的,很简单

2. 功能要求
   - 完成初始化的数据库连接(这部分功能我会提供函数),引导用户登陆打印欢迎页面
   - 在客户登陆时保存userid,authority方便在后面的过程中调用,避免用户重复输入
   - 加入权限识别,后台管理仅authority为0的人能够进行
   - 注意做好菜单分级,建议一级菜单为三部分,书籍查询,书籍借阅,管理员操作,在此处直接禁止非管理员选择3
   - 注意应用应当可以被反复使用,不会在执行完某函数后关闭
3. 自行比对Service中的内容,确保所有功能的实现



## 其他注意事项

1. 这里我贴一下三张表的格式

   ![image](https://s1.imagehub.cc/images/2024/11/29/bc99c8734be3ffa49cbd783285bb7eb1.png)

![image](https://s1.imagehub.cc/images/2024/11/29/56a3da7ec46baadc76bc52a026f63635.png)

![image](https://s1.imagehub.cc/images/2024/11/29/a5c6f42a7a68d4b76c792ece2bda4aa7.png)

book表少了一个position,无需在意,我已经补上了

2. 说一下开发要求
   - 命名一定要规范,我上面的函数只规定了具体的接口函数,但要完成项目应当还需要一些工具函数,创建时命名尽量规范,mapper层每个单词首字母大写,Service层使用驼峰命名法
   - 开头已经创建了对应的远程仓库,写完后提交在远程即可,注意,提交时不要直接提交到主分支,一人开一条分支,完成自己的部分后再合并分支,每次commit时说明提交的内容
   - mapper层和Service层的头文件与项目架构我已完成,每一层我都单独建了对应的目录,请在对应的目录内完成
   - 虽然具体实现没有完成,但是对应的声明已经完成,可以直接调用,方便所有人的编写
   - 这是我边想边写完成的,应该有接口对不上的问题,发现返回值不对劲和我说,因为用c处理数据库我也是第一次写

这些东西的实现都算不上复杂,但加起来我估计应该没几个比咱复杂的了,再上更复杂的不是不行,但感觉每必要

