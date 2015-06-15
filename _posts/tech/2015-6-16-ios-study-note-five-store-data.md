---
layout: post
category : Development
title : ios学习系列5--应用程序数据存储
tagline: "ios学习笔记"
tags : [ios,开发]
---
{% include JB/setup %}

本篇是第五部分，关于应用程序数据存储的一些内容。

####1. ios常用的数据存储方式

1. XML属性列表（plist）归档
2. Preference(偏好设置)
3. NSKeyedArchiver归档(NSCoding)
4. SQLite3
5. Core Data

####2. 沙盒

每个iOS应用都有自己的应用沙盒(应用沙盒就是文件系统目录)，与其他文件系统隔离。应用数据必须待在自己的沙盒里，其他应用不能访问该沙盒。目录结构如下：

   ![image](./images/ios_10.png)

1. 应用程序包：(上图中的Layer)包含了所有的资源文件和可执行文件。
2. Documents：保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏存档保存在该目录。
3. tmp：保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会备份该目录。
4. Library/Caches：保存应用运行时生成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大、不需要备份的非重要数据。
5. Library/Preference：保存应用的所有偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置信息。iTunes同步设备时会备份该目录。

目录路径获取：

1. 应用沙盒根目录：**NSString *home = NSHomeDirectory();**
2. Documents：

        // 方式1
		NSString *home = NSHomeDirectory();
		NSString *documents = [home stringByAppendingPathComponent:@"Documents"];
		
		// 方式2，不建议使用，因为新版本的操作系统可能会修改目录名，利用NSSearchPathForDirectoriesInDomains函数
		// NSUserDomainMask 代表从用户文件夹下找
		// YES 代表展开路径中的波浪字符“~”
		NSArray *array =  NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, NO);
		// 在iOS中，只有一个目录跟传入的参数匹配，所以这个集合里面只有一个元素
		NSString *documents = [array objectAtIndex:0];

3. tmp：NSString *tmp = NSTemporaryDirectory();
4. Library/Caches：(跟Documents类似的2种方法)

		利用沙盒根目录拼接”Caches”字符串
		利用NSSearchPathForDirectoriesInDomains函数(将函数的第1个参数改为：NSCachesDirectory即可)

5. Library/Preference：通过NSUserDefaults类存取该目录下的设置信息

####3. 属性列表
		
属性列表是一种XML格式的文件，拓展名为plist。如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，就可以使用writeToFile:atomically:方法直接将对象写到属性列表文件中。

字典写入文件：

    // 将一个NSDictionary对象归档到一个plist属性列表中
    // 将数据封装成字典
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    [dict setObject:@"母鸡" forKey:@"name"];
    [dict setObject:@"15013141314" forKey:@"phone"];
    [dict setObject:@"27" forKey:@"age"];
    // 将字典持久化到Documents/stu.plist文件中
    [dict writeToFile:path atomically:YES];

从文件中读取字典：

	// 读取属性列表，恢复NSDictionary对象
	// 读取Documents/stu.plist的内容，实例化NSDictionary
	NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:path];

####4. 偏好设置

许多ios应用支持使用偏好设置，如保存用户名，密码，程序设置等等。ios提供了一套标准的解决方案来为应用加入偏好设置。每个应用都有NSUserDefaults实例，通过它来存取偏好设置：

    // 保存偏好设置
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    [defaults setObject:@"cofcool" forKey:@"username"];
    [defaults setFloat:18.0f forKey:@"age"];
    [defaults setBool:YES forKey:@"auto_login"];
    
    // 读取偏好设置
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    NSString *username = [defaults stringForKey:@"username"];
    float age = [defaults floatForKey:@"age"];
    BOOL autoLogin = [defaults boolForKey:@"auto_login"];

需要注意的是，UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。为了避免偏好设置数据丢失，可以通过调用synchornize方法强制写入：

    [defaults synchornize];


####5. 归档

如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，可以直接用NSKeyedArchiver进行归档和恢复。对象只有遵守了NSCoding协议才可以使用归档。NSCoding协议的方法：

   * encodeWithCoder:
	每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的每个实例变量，可以使用encodeObject:forKey:方法归档实例变量。
   * initWithCoder:
	每次从文件中恢复(解码)对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用decodeObject:forKey方法解码实例变量。

   * 归档一个数组对象：

        // 归档
		NSArray *array = [NSArray arrayWithObjects:@”a”,@”b”,nil];
		[NSKeyedArchiver archiveRootObject:array toFile:path];
		
		// 恢复
		NSArray *array = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
  
  * 归档对象
  
     1. 定义Person类
      
            // Person.h
			@interface Person : NSObject<NSCoding>
			@property (nonatomic, copy) NSString *name;
			@property (nonatomic, assign) int age;
			@property (nonatomic, assign) float height;
			@end
			
			// Person.m
			@implementation Person
			- (void)encodeWithCoder:(NSCoder *)encoder {
				[encoder encodeObject:self.name forKey:@"name"];
			    [encoder encodeInt:self.age forKey:@"age"];
			    [encoder encodeFloat:self.height forKey:@"height"];
			}
			- (id)initWithCoder:(NSCoder *)decoder {
				self.name = [decoder decodeObjectForKey:@"name"];
				self.age = [decoder decodeIntForKey:@"age"];
				self.height = [decoder decodeFloatForKey:@"height"];
				return self;
			}
			- (void)dealloc {
				[super dealloc];
				[_name release];
			}
			@end

    2. 归档，恢复:
    
			// 归档(编码)
			Person *person = [[[Person alloc] init] autorelease];
			person.name = @"cofcool";
			person.age = 27;
			person.height = 1.83f;
			[NSKeyedArchiver archiveRootObject:person toFile:path];
			// 恢复(解码)
			Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:path];

     如果父类也遵守了NSCoding协议，请注意：应该在encodeWithCoder:方法中加上一句**[super encodeWithCode:encode]**确保继承的实例变量能被编码，即能被归档。另外应该在initWithCoder:方法中加上一句self = **[super initWithCoder:decoder]**确保继承的实例变量能被解码，即能被恢复。

  * NSData
  
    1. 使用archiveRootObject:toFile:方法可以将一个对象直接写入到一个文件中，但有时候可能想将多个对象写入到同一个文件中，那么就要使用NSData来进行归档对象。NSData可以为一些数据提供临时存储空间，以便随后写入文件，或者存放从磁盘读取的文件内容，可以使用[NSMutableData data]创建可变数据空间。

    2. 对象数据处理:
    
			//1. 归档（编码）
			// 新建一块可变数据区
			NSMutableData *data = [NSMutableData data];
			// 将数据区连接到一个NSKeyedArchiver对象
			NSKeyedArchiver *archiver = [[[NSKeyedArchiver alloc] initForWritingWithMutableData:data] autorelease];
			// 开始存档对象，存档的数据都会存储到NSMutableData中
			[archiver encodeObject:person1 forKey:@"person1"];
			[archiver encodeObject:person2 forKey:@"person2"];
			// 存档完毕(一定要调用这个方法)
			[archiver finishEncoding];
			// 将存档的数据写入文件
			[data writeToFile:path atomically:YES];
			
			// 2. 恢复
			// 从文件中读取数据
			NSData *data = [NSData dataWithContentsOfFile:path];
			// 根据数据，解析成一个NSKeyedUnarchiver对象
			NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
			Person *person1 = [unarchiver decodeObjectForKey:@"person1"];
			Person *person2 = [unarchiver decodeObjectForKey:@"person2"];
			// 恢复完毕
			[unarchiver finishDecoding];

    3. 利用归档实现深复制

			// 比如对一个Person对象进行深复制
			// 临时存储person1的数据
			NSData *data = [NSKeyedArchiver archivedDataWithRootObject:person1];
			// 解析data，生成一个新的Person对象
			Student *person2 = [NSKeyedUnarchiver unarchiveObjectWithData:data];
			// 分别打印内存地址
			NSLog(@"person1:0x%x", person1); // person1:0x7177a60
			NSLog(@"person2:0x%x", person2); // person2:0x7177cf0

####5. SQLite3

SQL语句分类：数据定义语句，数据操作语句，数据查询语句。

创建表：

	create table if not exists 表名 (字段名1 字段类型1, 字段名2 字段类型2, …);

SQLite3是一款开源的嵌入式关系型数据库，可移植性好、易使用、内存开销小。并且它是无类型的，意味着你可以保存任何类型的数据到任意表的任意字段中。为了保证可读性建议把字段类型加上，SQLite3常用的5种数据类型：text、integer、float、boolean、blob。要想使用SQLite3，首先要添加库文件libsqlite3.dylib和导入主头文件。

SQLite将数据划分为以下几种存储类型：integer : 整型值；real : 浮点值；text : 文本字符串；blob : 二进制数据（比如文件）。实际上SQLite是无类型的，如果声明为integer类型，还是能存储字符串文本（主键除外）。但是为了容易识别和阅读，建议加上字段类型。

良好的数据库编程规范应该要保证每条记录的唯一性，为此，增加了主键约束
也就是说，每张表都必须有一个主键，用来标识记录的唯一性。主键设计原则：主键应当是对当前用户没有意义的；永远不要更新主键；主键不应包含动态变化的数据；主键应由计算机自动生成。在创表的时候用primary key声明一个主键。利用外键约束可以用来建立表与表之间的联系，外键的一般情况是：一张表的某个字段，引用着另一张表的主键字段。

表连接：内连接：inner join 或者 join  （显示的是左右表都有完整字段值的记录）；左外连接：left outer join （保证左表数据的完整性）。




  1. 创建打开关闭数据库
  
		创建或打开数据库
		// path为：~/Documents/person.db
		sqlite3 *db;
		int result = sqlite3_open([path UTF8String], &db); 
		代码解析：
		sqlite3_open()将根据文件路径打开数据库，如果不存在，则会创建一个新的数据库。如果result等于常量SQLITE_OK，则表示成功打开数据库
		sqlite3 *db：一个打开的数据库实例
		数据库文件的路径必须以C字符串(而非NSString)传入
		关闭数据库：sqlite3_close(db);

  2. 执行SQL语句
  
		<1> 执行创表语句
			char *errorMsg;  // 用来存储错误信息
			char *sql = "create table if not exists t_person(id integer primary key autoincrement, name text, age integer);";
			int result = sqlite3_exec(db, sql, NULL, NULL, &errorMsg);
			
			代码解析：
			sqlite3_exec()可以执行任何SQL语句，比如创表、更新、插入和删除操作。但是一般不用它执行查询语句，因为它不会返回查询到的数据
			sqlite3_exec()还可以执行的语句：
			1. 开启事务：begin transaction;
			2. 回滚事务：rollback;
			3. 提交事务：commit;
		
		<2> 带占位符插入数据
			char *sql = "insert into t_person(name, age) values(?, ?);";
			sqlite3_stmt *stmt;
			if (sqlite3_prepare_v2(db, sql, -1, &stmt, NULL) == SQLITE_OK) {
				sqlite3_bind_text(stmt, 1, "母鸡", -1, NULL);
				sqlite3_bind_int(stmt, 2, 27);
			}
			if (sqlite3_step(stmt) != SQLITE_DONE) {
				NSLog(@"插入数据错误");
			}
			sqlite3_finalize(stmt);
			代码解析：
			sqlite3_prepare_v2()返回值等于SQLITE_OK，说明SQL语句已经准备成功，没有语法问题。
			sqlite3_bind_text()：大部分绑定函数都只有3个参数
			第1个参数是sqlite3_stmt *类型
			第2个参数指占位符的位置，第一个占位符的位置是1，不是0
			第3个参数指占位符要绑定的值
			第4个参数指在第3个参数中所传递数据的长度，对于C字符串，可以传递-1代替字符串的长度
			第5个参数是一个可选的函数回调，一般用于在语句执行后完成内存清理工作
			sqlite_step()：执行SQL语句，返回SQLITE_DONE代表成功执行完毕
			sqlite_finalize()：销毁sqlite3_stmt *对象
			
		<3>查询数据
			char *sql = "select id,name,age from t_person;";
			sqlite3_stmt *stmt;
			if (sqlite3_prepare_v2(db, sql, -1, &stmt, NULL) == SQLITE_OK) {
				while (sqlite3_step(stmt) == SQLITE_ROW) {
					int _id = sqlite3_column_int(stmt, 0);
					char *_name = (char *)sqlite3_column_text(stmt, 1);
					NSString *name = [NSString stringWithUTF8String:_name];
					int _age = sqlite3_column_int(stmt, 2);
					NSLog(@"id=%i, name=%@, age=%i", _id, name, _age);
				}
			}
			sqlite3_finalize(stmt);
			
			代码解析
			sqlite3_step()返回SQLITE_ROW代表遍历到一条新记录
			sqlite3_column_*()用于获取每个字段对应的值，第2个参数是字段的索引，从0开始

---

FMDB是iOS平台的SQLite数据库框架，FMDB以OC的方式封装了SQLite的C语言API。FMDB的优点：使用起来更加面向对象，省去了很多麻烦、冗余的C语言代码；对比苹果自带的Core Data框架，更加轻量级和灵活；提供了多线程安全的数据库操作方法，有效地防止数据混乱。

FMDB核心类：

1. FMDatabase：一个FMDatabase对象就代表一个单独的SQLite数据库，用来执行SQL语句。

2. FMResultSet：使用FMDatabase执行查询后的结果集。

3. FMDatabaseQueue：用于在多线程中执行多个查询或更新，它是线程安全的。

具体使用：

1. 打开数据库

		// 通过指定SQLite数据库文件路径来创建FMDatabase对象
		FMDatabase *db = [FMDatabase databaseWithPath:path];
		if (![db open]) {
		    NSLog(@"数据库打开失败！");
		}
	文件路径有三种情况:
	
	1. 具体文件路径：如果不存在会自动创建。
	
	2. 空字符串@""：会在临时目录创建一个空的数据库,当FMDatabase连接关闭时，数据库文件也被删除。
	
	3. nil：会创建一个内存中临时数据库，当FMDatabase连接关闭时，数据库会被销毁。

2. 执行更新

   在FMDB中，除查询以外的所有操作，都称为“更新”，create、drop、insert、update、delete等。

		// 使用executeUpdate:方法执行更新
		- (BOOL)executeUpdate:(NSString*)sql, ...
		- (BOOL)executeUpdateWithFormat:(NSString*)format, ...
		- (BOOL)executeUpdate:(NSString*)sql withArgumentsInArray:(NSArray *)arguments
		
		// 示例
		[db executeUpdate:@"UPDATE t_student SET age = ? WHERE name = ?;", @20, @"Jack"]

3. 执行查询

		// 查询方法
		- (FMResultSet *)executeQuery:(NSString*)sql, ...
		- (FMResultSet *)executeQueryWithFormat:(NSString*)format, ...
		- (FMResultSet *)executeQuery:(NSString *)sql withArgumentsInArray:(NSArray *)arguments
		
		// 查询数据
		FMResultSet *rs = [db executeQuery:@"SELECT * FROM t_student"];
		
		// 遍历结果集
		while ([rs next]) {
		    NSString *name = [rs stringForColumn:@"name"];
		    int age = [rs intForColumn:@"age"];
		    double score = [rs doubleForColumn:@"score"];
		}

4. FMDatabaseQueue

   FMDatabase这个类是线程不安全的，如果在多个线程中同时使用一个FMDatabase实例，会造成数据混乱等问题。为了保证线程安全，FMDB提供方便快捷的FMDatabaseQueue类。

		// FMDatabaseQueue的创建
		FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:path];
		
		// 简单使用
		[queue inDatabase:^(FMDatabase *db) {
		    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jack"];
		    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Rose"];
		    [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jim"];
		    
		    FMResultSet *rs = [db executeQuery:@"select * from t_student"];
		    while ([rs next]) {
		        // …
		    }
		}];

		// 使用事务
		[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
            [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jack"];
            [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Rose"];
            [db executeUpdate:@"INSERT INTO t_student(name) VALUES (?)", @"Jim"];
            
            FMResultSet *rs = [db executeQuery:@"select * from t_student"];
            while ([rs next]) {
                // …
            }
        }];
        
        // 事务回滚
        *rollback = YES;



####6. Core Data

Core Data框架提供了对象-关系映射(ORM)的功能，即能够将OC对象转化成数据，保存在SQLite3数据库文件中，也能够将保存在数据库中的数据还原成OC对象。在此数据操作期间，不需要编写任何SQL语句。使用此功能，要添加CoreData.framework和导入主头文件<CoreData/CoreData.h>。
		
![image](./images/ios_11.png)

  1. 模型文件
  
    在Core Data，需要进行映射的对象称为实体(entity)，而且需要使用Core Data的模型文件来描述应用的所有实体和实体属性。这里以Person和Card(身份证)2个实体为例子，先看看实体属性和之间的关联关系：
    ![image](./images/ios_12.png)
    
    创建模型文件
    
    ![image](./images/ios_13.png)
    
    ![image](./images/ios_14.png)
    
    ![image](./images/ios_15.png)
    
  2. 通过Core Data从数据库取出的对象，默认情况下都是NSManagedObject对象,NSManagedObject的工作模式有点类似于NSDictionary对象，通过键-值对来存取所有的实体属性:setValue:forKey: 存储属性值(属性名为key);valueForKey: 获取属性值(属性名为key)。

  3. Core Data主要对象
  
    ![image](./images/ios_16.png)

  4. 搭建Core Data上下文环境：
  
		// 从应用程序包中加载模型文件
		NSManagedObjectModel *model = [NSManagedObjectModel mergedModelFromBundles:nil];
		
		// 传入模型，初始化NSPersistentStoreCoordinator
		NSPersistentStoreCoordinator *psc = [[[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model] autorelease];
		
		// 构建SQLite文件路径
		NSString *docs = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
		NSURL *url = [NSURL fileURLWithPath:[docs stringByAppendingPathComponent:@"person.data"]];
		
		// 添加持久化存储库，这里使用SQLite作为存储库
		NSError *error = nil;
		NSPersistentStore *store = [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:url options:nil error:&error];
		if (store == nil) { // 直接抛异常
			[NSException raise:@"添加数据库错误" format:@"%@", [error localizedDescription]];
		}
		
		// 初始化上下文，设置persistentStoreCoordinator属性
		NSManagedObjectContext *context = [[NSManagedObjectContext alloc] init];
		context.persistentStoreCoordinator = psc;
		// 用完之后，还是要[context release];

		// 传入上下文，创建一个Person实体对象
		NSManagedObject *person = [NSEntityDescription insertNewObjectForEntityForName:@"Person" inManagedObjectContext:context];
		
		// 设置简单属性
		[person setValue:@"MJ" forKey:@"name"];
		[person setValue:[NSNumber numberWithInt:27] forKey:@"age"];
		
		// 传入上下文，创建一个Card实体对象
		NSManagedObject *card = [NSEntityDescription insertNewObjectForEntityForName:@"Card" inManagedObjectContext:context];
		[card setValue:@"4414241933432" forKey:@"no"];
		
		// 设置Person和Card之间的关联关系
		[person setValue:card forKey:@"card"];
		
		// 利用上下文对象，将数据同步到持久化存储库
		NSError *error = nil;
		BOOL success = [context save:&error];
		if (!success) {
			[NSException raise:@"访问数据库错误" format:@"%@", [error localizedDescription]];
		}
		
		// 如果是想做更新操作：只要在更改了实体对象的属性后调用[context save:&error]，就能将更改的数据同步到数据库
		
		// 初始化一个查询请求
		NSFetchRequest *request = [[[NSFetchRequest alloc] init] autorelease];
		
		// 设置要查询的实体
		NSEntityDescription *desc = [NSEntityDescription entityForName:@"Person" inManagedObjectContext:context];
		
		// 设置排序（按照age降序）
		NSSortDescriptor *sort = [NSSortDescriptor sortDescriptorWithKey:@"age" ascending:NO];
		request.sortDescriptors = [NSArray arrayWithObject:sort];
		
		// 设置条件过滤(name like '%Itcast-1%')
		NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name like %@", @"*Itcast-1*"];
		request.predicate = predicate;
		
		// 执行请求
		NSError *error = nil;
		NSArray *objs = [context executeFetchRequest:request error:&error];
		if (error) {
			[NSException raise:@"查询错误" format:@"%@", [error localizedDescription]];
		}
		
		// 遍历数据
		for (NSManagedObject *obj in objs) {
			NSLog(@"name=%@", [obj valueForKey:@"name"]);
		}
		
		// 传入需要删除的实体对象
		[context deleteObject:managedObject];
		
		// 将结果同步到数据库
		NSError *error = nil;
		[context save:&error];
		if (error) {
			[NSException raise:@"删除错误" format:@"%@", [error localizedDescription]];
		}

  5. Core Data的延迟加载
 
    Core Data不会根据实体中的关联关系立即获取相应的关联对象。比如通过Core Data取出Person实体时，并不会立即查询相关联的Card实体；当应用真的需要使用Card时，才会查询数据库，加载Card实体的信息。

  6. 创建NSManagedObject的子类
  
    默认情况下，利用Core Data取出的实体都是NSManagedObject类型，能够利用键-值对来存取数据。但是一般情况下，实体在存取数据的基础上，有时还需要添加一些业务方法来完成一些其他任务，那么就必须创建NSManagedObject的子类。
    
    ![image](./images/ios_17.png)
    
    ![image](./images/ios_18.png)
    
    ![image](./images/ios_19.png)

    生成一个Person实体对象：
    
		Person *person = [NSEntityDescription insertNewObjectForEntityForName:@"Person" inManagedObjectContext:context];
		person.name = @"cofcool";
		person.age = [NSNumber numberWithInt:27];
		
		Card *card = [NSEntityDescription insertNewObjectForEntityForName:@”Card" inManagedObjectContext:context];
		card.no = @”4414245465656";
		person.card = card;

