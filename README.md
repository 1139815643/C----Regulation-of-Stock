# C----Regulation-of-Stock
A simple refulation of stock
//#define _CRT_SECURE_NO_WARNINGS
//#include <iostream>
//#include"Interface.h"
//using namespace std;
////void test_read_pwd() {
////	cout << isalnum('a') << isalnum('1') << isalnum(12);
////	string test = readPassword();
////	cout << "this is" << test << endl;
////}
////void parse_test() {
////	string test = "oijweof,ewifjoas,ewfiojaosidjfo";
////	vector<string> a = parse(',', test);
////	for (int i = 0; i < a.size(); i++) cout << a[i] << endl;
////}
//int main() {
//	while (1) {
//		Interface inst;
//		inst.~Interface();
//		system("pause");
//	}
//}
#define _CRT_SECURE_NO_WARNINGS
#include<cstdlib>
#include<cstring>
#include<cstdio>   //FILE 头文件 
#include<iostream>
#include<string>
#include<vector>
#include<unordered_map>
#include<conio.h>
#include<iomanip>
using namespace std;
//一些工具
bool examPwd(string pwd){
	if (pwd.size() < 8)return false;//密码必须长于8位
									//这里可以添加新的规则
	return true;
}
vector<string> parse(char a, string word) {   //   sting类 vector 把word分割 ,放到ret中 
	string temp; vector<string>ret;
	for (int i = 0; i < word.size(); i++) {
		if (word[i] != a)temp += word[i];
		else ret.push_back(temp), temp.clear();  //push_back 添加一个元素到向量末尾  clear() 清空向量 
	}
	if (temp.size())ret.push_back(temp);
	return ret;
}
string readPassword() {//从stdin读取密码,不回显
	char buffer = '.'; string ret = "";
	while (1) {
		buffer = getch();//*************编译不过的话用getch()****************
		if (!isalnum(buffer))break;  //isalnum()判断是否为字母或数字 
		ret += buffer; putchar('*');  //默认重载 +=     putchar() 向终端输出字符 
	}
	return ret;
}
inline bool is_base64(unsigned char c) {
	return (isalnum(c) || (c == '+') || (c == '/'));
}

string readAll(FILE * f) {
	char buffer[2048]; memset(buffer, 0, sizeof buffer);  //buffer(数组）  0（赋给buffer的值） sizeof buffer（实际长度）多次清空数组,很方便的清空结构类型的变量 
	fgets(buffer, 2047, f);  //从文件流中读取指定个数的字符，buffer用于保存。 
	return buffer;
}
/////////////////////////////////
struct StockPool;
class UserManager;
StockPool *stock_pool;
UserManager *user_manager;
//struct StockPool stock_pool;
//class UserManager user_manager;
///////////////////////////////
struct Stock {
	std::string id, name; int remaining;  //   股票代码  名称   发行股数 
	bool stopped;//是否停止交易
	std::vector<float>price_log;//数组，记录了历史价格
	Stock() { stopped = 0; }
	Stock(std::string id, std::string n, int a, std::vector<float> p) :id(id), name(n), remaining(a), price_log(p) { stopped = 0; }
	float getCurrentPrice() {
		return price_log[price_log.size() - 1];    // 股票的当前价格 
	}
	float getHistoryPrice(int days_cnt) {
		return price_log[price_log.size() - 1 - days_cnt];    //股票的历史价格，size()返回当前元素个数 
	}
	float getRise() {
		return (getCurrentPrice() - getHistoryPrice(1)) / getHistoryPrice(1);   //计算涨幅 
	}
	void changeIdName(std::string i, std::string n) {     //改变id name 
		id = i, name = n;
	}
	string toString() {
		string ret = id + ',' + name + ',' + to_string(remaining) + ',' + char('0' + stopped);
		for (int i = 0; i < price_log.size(); i++)
			ret += ',' + to_string(price_log[i]);   //to_sting()函数，把价格和，相加作为字符串。 
		return ret + ';';     //“+”的默认重载 
	}
};
struct StockPool {    //股票池，记录股票的基本信息 
	std::unordered_map<std::string, struct Stock>pool;    // 关联容器（哈希桶） key:键值，用string参照stock达到快速访问，通过【】实现，以键值为参数访问映射值 
	FILE *stock_info, *buffer;         //定义文件指针 * 
	StockPool() {//构造函数 
		stock_info = fopen("stock.data", "r+");   //文件操作函数，对字符串进行操作，正文读写方式打开。 
		buffer = fopen("Sbuf", "w+");    //文件操作函数，正文文件读写方式打开。 

		string temp = readAll(stock_info);  //读取整个文件的基本信息 
		readFromString(temp);
	}

	void readFromString(std::string &temp) {    //从字符串中读取所有股票信息并写入内存 
		vector<string> all_user = parse(';', temp);  //给容器赋值   
		for (int i = 0; i < all_user.size(); i++) {
			vector<string> this_stock = parse(',', all_user[i]);  //
			//根据格式，此处this_user[0]为用户名，this_user[1]为密码
			// if(this_user[2].size()>1)throw exception("input error!");
			vector<float>temp;
			for (int i = 4; i < this_stock.size(); i++)temp.push_back(atof(this_stock[i].c_str()));  // atof()把字符串转化成浮点数，c-str()转换成字符数组 
			pool[this_stock[0]] =
				Stock(this_stock[0], this_stock[1], atoi(this_stock[2].c_str()), temp);       //atoi()把字符串转化为整数，关联容器映射关系建立，key值为用户名 
				// 通过 this_stock快速索引到对应的stock，调用stock构造函数，实现把股票的基本信息写入内存。 
		}
	}
	void writeBuffer(Stock a) {
		fprintf(buffer, a.toString().c_str());  //向文件里写入字符串信息 
	}
	void readBuffer() {
		string temp = readAll(buffer); //从文件中读取数据 
		readFromString(temp);     //写入内存 
	}
	void fullUpdate() {  //将内存中的所有信息读入内存 
		string temp;
		for (unordered_map<string, Stock>::iterator i = pool.begin();  //返回第一个元素为迭代器起始 
			i != pool.end(); i++)  //遍历内存中股票信息
			temp += i->second.toString();  //利用迭代器找到映射的stock（）并调用stock成员函数 
		fclose(stock_info); stock_info = fopen("stock.data", "w");  //清空文件 
		fprintf(stock_info, temp.c_str());//写入最终信息
		fclose(stock_info); stock_info = fopen("stock.data", "r+");
	}
	~StockPool() {
		fullUpdate();

		fclose(stock_info);
		fclose(buffer);
	}
};
///////////////////////////////
class User {
	std::string username, password;   //用户名  密码 
	bool is_admin; //是否管理员 

public:
	std::unordered_map<std::string, int>owned_stocks;   //关联容器，key值 关联持股数量 
	float balance;
	User() {}
	User(std::string u, std::string p, bool id) :username(u), password(p), is_admin(id) {}
	bool checkPwd(std::string p) { return p == password; } //验证密码 
	bool editPwd(std::string p) { if (!examPwd(p))return false; password = p; return true; } //修改密码 

	bool buy(std::string id, int amount) {
		if (stock_pool->pool.find(id) == stock_pool->pool.end())return false;// 利用stock_pool指针调用容器find函数，查找股票名称 
		Stock *target = &stock_pool->pool[id];//避免多次寻址
		if (target->stopped)return false;//股票停止交易
		if (target->remaining < amount)return false;//市场中的股票不够用户买的
		if (target->getCurrentPrice()*amount > balance)return false;//账户余额不足
		target->remaining -= amount;
		owned_stocks[target->id] += amount;   //持股数量变化 
		balance -= target->getCurrentPrice()*amount; //余额变化 

		stock_pool->writeBuffer(stock_pool->pool[id]);  //向文件里写入变化 
		//user_manager->writeBuffer(*this);
		return true;//购买成功
	}
	bool sell(std::string id, int amount) {// 销售 
		if (stock_pool->pool.find(id) == stock_pool->pool.end())return false;//无此股票
		Stock *target = &stock_pool->pool[id]; //定义stock指针指向通过map容器找到的stock 
		if (target->stopped)return false;
		if (owned_stocks[id] < amount)return false;//账户中没有这么多股票可卖
		balance += target->getCurrentPrice()*amount;
		owned_stocks[id] -= amount;
		target->remaining += amount;
		if (owned_stocks[id] == 0)owned_stocks.erase(id);//账户中已无此类股票，清除 

		stock_pool->writeBuffer(stock_pool->pool[id]);
		//user_manager->writeBuffer(*this);//应说明要求，写缓存，单文件由于前向声明无法实现
		return true;//卖出成功
	}
	//*************管理员操作****************
	bool addStock(std::string id, std::string name, float init_price, int amount) {
		if (!is_admin)return false;
		vector<float>temp; temp.push_back(init_price);
		stock_pool->pool[id] = Stock(id, name, amount, temp); //更新信息  此时主要更新temp的价格 
		return true;
	}
	bool editStock(std::string id, std::string new_name, std::string new_id) {//修改股票名，股票代码
		if (!is_admin)return false;
		if (stock_pool->pool.find(id) == stock_pool->pool.end())return false;//没有代码为id的股票
		stock_pool->pool.erase(id);  //撤销股票名称 
		stock_pool->pool[new_id] = stock_pool->pool[id];  
		/*stock_pool.pool[new_id].id = new_id;
		stock_pool.pool[new_id].name = new_name;*/
		stock_pool->pool[new_id].changeIdName(new_id, new_name);
		return true;
	}
	bool toggleStock(std::string id) {
		if (!is_admin)return false;
		stock_pool->pool[id].stopped ^= 1; //停止交易 ，解挂股票 
		return true;
	}
	//*****************************************

	std::string toString() {
		string ret = username + ',' + password + ',' + char(is_admin + '0');
		for (unordered_map<string, int>::iterator i = owned_stocks.begin();
			i != owned_stocks.end(); i++)
			ret += ',' + i->first + ',' + to_string(i->second);
		ret += ';';
		return ret;
	}
	~User() {}
};
class UserManager {  //用户管理 
	std::unordered_map<std::string, class User> pool;  //key值：string  参照user  
	FILE *user_info, *buffer;
	void readFromString(std::string &temp) { //写入用户的信息到内存中 
		vector<string> all_user = parse(';', temp);
		for (int i = 0; i < all_user.size(); i++) {
			vector<string> this_user = parse(',', all_user[i]);
			//根据格式，此处this_user[0]为用户名，this_user[1]为密码
			// if(this_user[2].size()>1)throw exception("input error!");
			pool[this_user[0]] =
				User(this_user[0], this_user[1], this_user[2][0] - '0');
			for (int i = 3; i < this_user.size(); i += 2)
				pool[this_user[0]].owned_stocks[this_user[i]] = atoi(this_user[i + 1].c_str());
		}
	}
public:
	UserManager() {
		user_info = fopen("user.data", "r+");// 打开文件，进行写操作 
		buffer = fopen("Ubuf", "w+"); //创建缓存区 

		//读取以前的用户
		string temp = readAll(user_info);
		readFromString(temp);
	}

	User *getUser(std::string username) { //获取用户名 
		if (pool.find(username) != pool.end())
			return &pool[username];
		else return 0;
	}
	bool addUser(std::string username, std::string password, bool isAdmin) {
		if (pool.find(username) != pool.end())return false;//用户名被占用
		pool[username] = User(username, password, isAdmin);//通过usename索引到建立映射关系的user,调用其构造函数 
		writeBuffer(pool[username]);//向文件中写入信息 
		return true;
	}
	bool deleteUser(std::string username) {
		if (pool.find(username) == pool.end())return false;
		//pool[username].~User();
		pool.erase(username);  //删除容器中的用户 
		return true;
	}

	void fullUpdate() {//完整更新存储文件（编码存储）
		string temp;
		for (unordered_map<string, User>::iterator i = pool.begin();
			i != pool.end(); i++)  //遍历内存中用户信息
			temp += i->second.toString();

		fclose(user_info); user_info = fopen("user.data", "w");
		fprintf(user_info, temp.c_str());//写入最终信息
		fclose(user_info); user_info = fopen("user.data", "r+");
	}
	void readBuffer() {
		string temp = readAll(buffer);
		readFromString(temp);  //对文件进行读写操作 
	}
	void writeBuffer(User a) { //写入缓存
		fprintf(buffer, a.toString().c_str());
	}

	~UserManager() {
		fullUpdate();

		fclose(user_info);
		fclose(buffer);
	}
};
///////////////////////////////
//su密码:1234567890
class SuperUser {
public:
	//设置当日价格,无法修改历史价格
	bool setStockPrice(std::string id, float p) {
		if (stock_pool->pool.find(id) == stock_pool->pool.end())return false;
		stock_pool->pool[id].price_log.push_back(p);  //添加价格p到vector的尾部 
		return true;
	}
	//注册
	class User *addUser(std::string username, std::string password, bool isAdmin) {
		if (user_manager->addUser(username, password, isAdmin))
			return user_manager->getUser(username);
		return 0;
	}
	//设置用户余额
	bool setBalance(std::string username, float b) {
		if (user_manager->getUser(username)) {
			user_manager->getUser(username)->balance = b;   //通过user_manager指针调用类use_mananger的成员函数，得到通过容器键值找到的用户类，设置用户余额。 
			return true;
		}
		return false;
	}
	//删除用户
	bool deleteUser(std::string username) {
		return user_manager->deleteUser(username);  //指针调用，用法同上 
	}
};
//////////////////////////////
class Interface {     //接口类 （操作以上全部类） 
	void suLogin() {     //超管登陆 
		system("cls");
		cout<<"输入su密码:"<<endl;
		string pwd; pwd = readPassword();
		if (pwd != "1234567890") {
			cout<<"密码不正确，程序重启。"<<endl;
			return;
		}
		char f = 0;
		SuperUser instance;

		while (1) {
			system("cls");
			cout<<"已登录su\n"<<endl;
			f = 0;
			cout<<"1.调整股票价格\n2.删除用户\n3.修改用户余额\n4.返回\n"<<endl;
			cout<<"请输入数字:"<<endl;
			while (!isalnum(f))f = getchar();
			while (f<'1' || f>'4') { cout<<"请输入正确数字:"<<endl; getchar(); f = getchar(); }
			string buffer1; float buffer3;
			switch (f) {
			case '1'://调整股票价格
				cout<<"请输入股票代码:"<<endl; getchar();
				cin >> buffer1;
				cout<<"请输入修改后价格:"<<endl;
				cin >> buffer3;
				if (instance.setStockPrice(buffer1, buffer3))cout<<"成功\n"<<endl;
				else cout<<"失败\n"<<endl;
				break;
			case '2'://删除用户
				cout<<"请输入要删除的用户名:"<<endl;
				cin >> buffer1;
				if (instance.deleteUser(buffer1))cout<<"成功\n"<<endl;
				else cout<<"失败，可能没有此用户\n"<<endl;
				break;
			case '3':
				cout<<"输入用户名:"; cin >> buffer1;
				cout<<"输入余额:"; cin >> buffer3;
				if (instance.setBalance(buffer1, buffer3))cout<<"成功\n"<<endl;
				else cout<<"查无此用户\n"<<endl;
				break;
			case '4':
				return;
			}
			system("pause");
		}
	}
	void userLogin() {
		string username, password; char f = 0;
		cout<<"请输入用户名:"; cin >> username;
		User *target = user_manager->getUser(username);
		cout<<"请输入密码:"<<endl; password = readPassword();
		if (target->checkPwd(password) == false) {
			cout<<"密码错误，重启程序"<<endl;
			return;
		}
		while (1) {
			system("cls");
			f = 0;
			printf("已登录%s\n", username.c_str());
			cout<<"1.修改密码\n2.查询股票\n3.买入股票\n4.卖出股票\n5.返回\n"<<endl;
			cout<<"管理员行为：\n6.添加股票\n7.修改股票代码、名称\n8.切换股票停售、在售状态\n"<<endl;
			cout<<"请输入一个指令:"<<endl;
			while (!isalnum(f))f = getchar();
			while (f<'1' || f>'8') { cout<<"请输入正确数字:"<<endl; getchar(); f = getchar(); }
			string buffer1, buffer2, buffer5; float buffer3; int buffer4;
			switch (f) {
			case '1'://修改密码
				cout<<"请输入新密码:"<<endl; cin >> password;
				if (target->editPwd(password))cout<<"成功\n";
				else cout<<"失败，密码不符合要求\n"<<endl;
				break;
			case '2'://查询股票
				query();
				break;

			case '3'://购买股票
				cout<<"输入股票代码:"; cin >> buffer1;
				cout<<"输入购买数量:"; cin >> buffer4;
				if (target->buy(buffer1, buffer4))cout<<"购买成功\n"<<endl;
				else cout<<"购买失败，有以下几种可能：\n股票代码有误\n股票关闭交易\n余额不足\n市场中股票不足\n";
				break;
			case '4'://出售股票
				cout<<"输入股票代码"; cin >> buffer1;
				cout<<"输入出售数量:"; cin >> buffer4;
				if (target->sell(buffer1, buffer4))printf("出售成功\n");
				else cout<<"出售失败，有以下几种可能：\n股票代码有误\n股票关闭交易\n拥有股票不足\n";
				break;

			case '5':return;//返回

			case '6'://新增股票
				cout<<"新股票代码:"; cin >> buffer1;
				cout<<"新股票名称:"; cin >> buffer2;
				cout<<"首发价格:"; cin >> buffer3;
				cout<<"发行数量:"; cin >> buffer4;
				if (target->addStock(buffer1, buffer2, buffer3, buffer4))
					cout<<"成功\n"<<endl;
				else cout<<"用户不是管理员，操作失败\n";
				break;

			case '7'://修改股票
				cout<<"股票代码:"; cin >> buffer1;
				cout<<"股票新代码(不填则为不修改):"; cin >> buffer2;
				cout<<"股票新名称(必须修改):"; cin >> buffer5;
				if (buffer2.size() == 0)buffer2 = buffer1;
				if (buffer5.size() == 0) {
					printf("输入错误\n");
					break;
				}
				if (target->editStock(buffer1, buffer5, buffer2))
					cout<<"成功\n";
				else cout<<"用户不是管理员，操作失败\n";
				break;
			case '8'://切换状态
				cout<<"股票代码:"; cin >> buffer1;
				if (target->toggleStock(buffer1))
					printf("成功\n");
				else cout<<"用户不是管理员，操作失败\n";
				break;
			}
			system("pause");
		}
	}
	void addUser() {         //创建用户 
		string username, password;
		SuperUser inst;
		cout<<"请输入用户名:";
		cin >> username;
		cout<<"请输入密码:";
		cin >> password;
		cout<<"是管理员么？(y/n)"<<endl; getchar(); char c = getchar(); getchar();
		if (c == 'y') {
			printf("su密码:");
			string pwd = readPassword();   //验证超级管理员 
			if (pwd != "1234567890") {
				cout<<"su密码错误\n";
				return;
			}
		}
		inst.addUser(username, password, c == 'y' ? 1 : 0);
	}
	void query() {
		system("cls");
		cout<<"欢迎来到查询系统\n";
		string id, name;
		cout<<"请输入股票代码（可以不完整）:"; cin >> id;
		vector<Stock*>list;
		for (unordered_map<std::string, Stock>::iterator i = stock_pool->pool.begin();
			i != stock_pool->pool.end(); i++)
			if (i->second.id.find(id) == 0)list.push_back(&i->second);//找出所有股票代码以id开头的股票

		if (list.size() == 0) {
			cout<<"未找到任何股票信息\n";
			return;
		}
		cout<<"找到了下列股票:\n";
		for (int i = 0; i < list.size(); i++)
			printf("%d . %s\n", i + 1, list[i]->name.c_str());
		cout<<"请选择股票:";
		int n = 0; cin >> n;

		printTable(list[n - 1]);
	}


	void printTable(struct Stock* i) {
		printf("---------------------------价格走势图-----------------------------\n");
		vector<float>temp = i->price_log;
		float max_price = 0, min_price = 99999999.9f;
		for (int i = 0; i < temp.size(); i++) {
			if (temp[i] < min_price)min_price = temp[i];
			if (temp[i] > max_price)max_price = temp[i];
		}
		int cnt = 0;
		for (float i = max_price; i >= min_price - 0.3; i -= 0.1) {
			if (cnt++ % 4 == 0)cout << setiosflags(ios::right) << setw(3) << i << setw(1) << '|';
			else cout << setw(4) << '|';
			for (int j = 0; j < temp.size(); j++)
				if (temp[j] - i < 0.001&&temp[j] - i>-0.001)cout << setiosflags(ios::right) << setw(3 * j + 1) << "*";
			cout << '\n';
		}
		printf("---");
		for (int i = 0; i < temp.size(); i++)printf("---");
		printf("\n");
		cout << "day 1";
		for (int i = 2; i <= temp.size(); i++)
			cout << setiosflags(ios::right) << setw(3) << i;
		printf("\n\n\n\n\n\n\n");
	}

public:
	Interface() {
		char f = 0;
		system("cls");
		cout<<"\n\n\n\n\n**********************WELCOME!***********************\n\n\n\n\n\n";
		cout<<"*********TO STOCK EXCHANGE SIMULATION PROGRAM!***********\n\n\n\n";
		cout<<"请选择功能:\n1.超级用户登陆\n2.登陆账户\n3.注册用户\n";
		cout<<"请输入数字:";
		while (!isalnum(f))f = getchar();
		while (f<'1' || f>'3') { cout<<"请输入有效数字:"; getchar(); f = getchar(); }
		getchar();
		switch (f) {
		case '1':suLogin(); break;
		case '2':userLogin(); break;
		case '3':addUser(); break;
		}
	}
	~Interface() {
		user_manager->fullUpdate();
		stock_pool->fullUpdate();
	}

};
int main() {
	user_manager = new UserManager;
	stock_pool = new StockPool;
	while (1) {
		Interface inst;
		inst.~Interface();
		system("pause");
	}
}
