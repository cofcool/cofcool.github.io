---
layout: post
category : Development
title : ios练习系列1--简单的搜索
tagline: "ios练习笔记"
tags : [ios]
---
{% include JB/setup %}

在学习ios的过程中，做了一些练习，总结如下。

在使用UITableView的场景中经常要用到搜索功能需要从许多行数据中搜索对应的内容。

搜索的过程大致如下：

1. 获取输入的关键字；
2. 把关键字和数组中存储的数据进行对比；
3. 如果数组中包含关键字，则把该数组的这条数据添加到搜索结果数组中；
4. 刷新表格，显示搜索结果

效果:

![image]({{ site.url }}/public/upload/images/ios_37.png)

![image]({{ site.url }}/public/upload/images/ios_38.png)


## 代码

搜索框使用UISearchBar控件，数据展示使用UITableView控件。

1. 创建两个数组：

		/**
		 *  数据来源
		 */
		@property (nonatomic,strong) NSArray *array;
		
		/**
		 *  储存符合搜索条件的对象数据
		 */
		@property (nonatomic,strong) NSMutableArray *searchArray;

2. 搜索代码：

		/**
		 *  开始搜索
		 *
		 *  @param searchText 搜索内容
		 */
		- (void)startSearch:(NSString *)searchText {
		    if (searchText.length > 0) {
		        [self.searchArray removeAllObjects];
		#warning 如何把数组内的字典转换为可以查询的数据
		        // 疯狂遍历数组，逐个查询
		        for (NSDictionary *dict in self.array) {
		            NSString *keyword = dict[@"keyword"];
		            // 是否包含指定关键字
		            if ([keyword containsString:searchText]) {
		                [self.searchArray addObject:dict];
		            }
		        }		        
		    }else   {
		        // 没有搜索文字时清空数组
		        [self.searchArray removeAllObjects];
		    }
		    
		    // 刷新数据
		    [self.tableView reloadData];
		}

3. 获取用户输入，调用搜索方法。searchBar的内容发生改变时，告诉代理内容发生改变，代理就会调用下述方法，在该方法中调用第二步的搜索方法。

		#pragma mark - 实现searchBar的代理方法
		
		/**
		 *  searchBar内容改变时调用
		 */
		- (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText {
		    
		    // 开始搜索
		    [self startSearch:searchText];
		}
		
3. 调用UITableView的数据源方法，设置cell数据，**searchArray**数组作为数据来源。

___

以上就是一个简单的搜索实现，不过还有很多问题，搜索能力很弱，另外也可以使用**NSPredicate**和正则表达式来实现字符匹配。