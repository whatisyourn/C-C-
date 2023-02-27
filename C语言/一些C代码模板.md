# 1.数组接收信息

- 保证能在收的地方后面加\0

```C
char buffer[6] = {0};
char *mystring(){
    char *s = "Hello world";
    for(int i = 0;i < (sizeof(buffer) - 1); i++){
    	buffer[i] = *(s+i);
    }
    return buffer;
}
```

# 2.简单的时间格式工具类

.h

```C++
#pragma once
#ifndef _SIMPLEDATETIMEFORMAT_H_
#define _SIMPLEDATETIMEFORMAT_H_
#include <string>

/**
 * 定义一个简单的时间格式工具类
 * 参考：https://zh.cppreference.com/w/cpp/io/manip/put_time
 */
class SimpleDateTimeFormat final
{
public:
	//************************************
	// Method:    format
	// FullName:  SimpleDateTimeFormat::format
	// Access:    public static 
	// Returns:   std::string 返回格式化后的字符串
	// Qualifier: 获取当前时间格式字符串
	// Parameter: const std::string & fmt 格式字符串，默认值%Y-%m-%d %H:%M:%S（对应格式如：2023-01-01 01:01:01）
	//************************************
	static std::string format(const std::string& fmt = "%Y-%m-%d %H:%M:%S");

	//************************************
	// Method:    formatWithMilli
	// FullName:  SimpleDateTimeFormat::formatWithMilli
	// Access:    public static 
	// Returns:   std::string 返回格式化后的字符串
	// Qualifier: 获取当前时间格式字符串，带毫秒时间获取
	// Parameter: const std::string & fmt 格式字符串，默认值%Y-%m-%d %H:%M:%S（对应格式如：2023-01-01 01:01:01）
	// Parameter: const std::string msDelim 毫秒与前部分的分割符，默认是空格
	//************************************
	static std::string formatWithMilli(const std::string& fmt = "%Y-%m-%d %H:%M:%S", const std::string msDelim = " ");
};

#endif // !_SIMPLEDATETIMEFORMAT_H_
```

```C++
/*
 Copyright Zero One Star. All rights reserved.

 @Author: awei
 @Date: 2023/02/24 15:45:36

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

	  https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
*/
#include "pch.h"
#include "../include/SimpleDateTimeFormat.h"
#include <chrono>
#include <sstream>
#include <iomanip>
//生成没有毫秒格式的
std::string SimpleDateTimeFormat::format(const std::string& fmt /*= "%Y-%m-%d %H:%M:%S"*/)
{
	// 获取当前时间
	auto now = std::chrono::system_clock::now();
	
	// 格式时间
	std::stringstream ss;
	auto tNow = std::chrono::system_clock::to_time_t(now);
	ss << std::put_time(std::localtime(&tNow), fmt.c_str());
	return ss.str();
}
//生成有毫秒格式的
std::string SimpleDateTimeFormat::formatWithMilli(const std::string& fmt /*= "%Y-%m-%d %H:%M:%S"*/, const std::string msDelim /*= " "*/)
{
	// 获取当前时间
	auto now = std::chrono::system_clock::now();
	
	// 格式化时间
	std::stringstream ss;
	auto tNow = std::chrono::system_clock::to_time_t(now);
	ss << std::put_time(std::localtime(&tNow), fmt.c_str());

	// 获取当前时间的秒数
	auto tSeconds = std::chrono::duration_cast<std::chrono::seconds>(now.time_since_epoch());
	// 获取当前时间的毫秒
	auto tMilli = std::chrono::duration_cast<std::chrono::milliseconds>(now.time_since_epoch());
	// 作差求出毫秒数
	auto ms = tMilli - tSeconds;
	ss << msDelim << std::setfill('0') << std::setw(3) << ms.count();
	return ss.str();
}

```

# 3.execl操作工具xnlt模板

`ExcelComponent.h`

```C++
#pragma once
#ifndef _EXCELCOMPONENT_H_
#define _EXCELCOMPONENT_H_
#include <xlnt/xlnt.hpp>
#include <string>
#include <vector>
/**
 * Excel组件
 */
class ExcelComponent
{
private:
	xlnt::workbook wb;
	xlnt::worksheet sheet;
	double rowHeight = 20;
	double colWidth = 20;
	//初始化工作薄
	void initWorkbook(const std::string& fileName = "");
	//创建Sheet
	void createSheet(const std::string& sheetName);
public:
	//设置行高
	void setRowHeight(double rowHeight);
	//设置列宽
	void setColWidth(double colWidth);
	//读取指定页签内容到容器,注意文件路径使用/
	std::vector<std::vector<std::string>> readIntoVector(const std::string& fileName, const std::string& sheetName);
	//保存指定页签内容到文件,注意文件路径使用/
	void writeVectorToFile(const std::string& fileName, const std::string& sheetName, const std::vector<std::vector<std::string>>& data);
};
#endif // _EXCELCOMPONENT_H_

```

`ExcelComponent.cpp`

```C++
#include "pch.h"
#include "../include/ExcelComponent.h"
#include "../include/CommonMacros.h"
#include <iostream>

void ExcelComponent::initWorkbook(const std::string& fileName /*= ""*/)
{
	wb = xlnt::workbook();
	if (!fileName.empty())
	{
		try
		{
			wb.load(fileName);
		}
		catch (xlnt::exception e)
		{
			std::cout << e.what() << std::endl;
		}
	}
}

void ExcelComponent::createSheet(const std::string& sheetName)
{
	sheet = wb.active_sheet();
	sheet.title(sheetName);
}

void ExcelComponent::setRowHeight(double rowHeight)
{
	this->rowHeight = rowHeight;
}

void ExcelComponent::setColWidth(double colWidth)
{
	this->colWidth = colWidth;
}

std::vector<std::vector<std::string>> ExcelComponent::readIntoVector(const std::string& fileName, const std::string& sheetName)
{
	std::vector<std::vector<std::string>> result;
	initWorkbook(fileName);
	if (!wb.contains(sheetName))
	{
		return result;
	}
	auto sheet = wb.sheet_by_title(sheetName);
	for (auto row : sheet.rows(false))
	{
		std::vector<std::string> aSingleRow;
		for (auto cell : row)
		{
			aSingleRow.push_back(cell.to_string());
		}
		result.push_back(aSingleRow);
	}
	return result;
}

void ExcelComponent::writeVectorToFile(const std::string& fileName, const std::string& sheetName, const std::vector<std::vector<std::string>>& data)
{
	initWorkbook();
	createSheet(sheetName);
	int row = 1;
	int col = 1;
	for (auto aSignRow : data)
	{
		col = 1;
		for (auto cellVal : aSignRow)
		{
			//设置单元格值
			sheet.cell(xlnt::cell_reference(col, row)).value(cellVal);
			//设置列宽度
			sheet.column_properties(col).custom_width = true;
			sheet.column_properties(col).width = colWidth;
			col++;
		}
		//设置行高度
		sheet.row_properties(row).custom_height = true;
		sheet.row_properties(row).height = rowHeight;
		row++;
	}

	//判断目录是否存在，不存在创建目录
	auto dir = fileName.substr(0, fileName.find_last_of("/") + 1);
	const size_t dirLen = dir.length();
	if (dirLen > MAX_DIR_LEN)
	{
		std::cout << "表格保存失败<文件路径太长>" << std::endl;
		return;
	}
	char tmpDirPath[MAX_DIR_LEN] = { 0 };
	for (size_t i = 0; i < dirLen; i++)
	{
		tmpDirPath[i] = dir[i];
		if (tmpDirPath[i] == '/')
		{
			if (ACCESS(tmpDirPath, 0) != 0)
			{
				if (MKDIR(tmpDirPath) != 0)
				{
					std::cout << "表格保存失败<创建文件失败>：" << tmpDirPath << std::endl;
					return;
				}
			}
		}
	}

	//保存到文件
	wb.save(fileName);
}
```

使用示例

```C++
#include "stdafx.h"
#include "ExcelComponent.h"
#include "CharsetConvertHepler.h"
using namespace std;

int main()
{
	//创建测试数据
	vector<vector<std::string>> data;
	stringstream ss;
	for (int i = 1; i <= 10; i++)
	{
		vector<std::string> row;
		for (int j = 1; j <= 5; j++)
		{
			ss.clear();
			ss
				<< CharsetConvertHepler::ansiToUtf8("单元格坐标：(") << i
				<< CharsetConvertHepler::ansiToUtf8(",") << j << ")";
			row.push_back(ss.str());
			ss.str("");
		}
		data.push_back(row);
	}

	//定义保存数据位置和页签名称
	std::string fileName = "./public/excel/1.xlsx";
	std::string sheetName = CharsetConvertHepler::ansiToUtf8("数据表");

	//保存到文件
	ExcelComponent excel;
	excel.writeVectorToFile(fileName, sheetName, data);

	//从文件中读取
	auto readData = excel.readIntoVector(fileName, sheetName);
	for (auto row : readData)
	{
		for (auto cellVal : row)
		{
			cout << CharsetConvertHepler::utf8ToAnsi(cellVal) << ",";
		}
		cout << endl;
	}
}
```

# 4.字符串编码转码工具类

`CharsetConvertHepler.h`

```C++
#pragma once
#ifndef _CHARSETCONVERTHEPLER_H_
#define _CHARSETCONVERTHEPLER_H_
#include <string>
/**
 * 字符串编码转码工具类
 */
class CharsetConvertHepler final
{
public:
	//************************************
	// Method:    unicodeToUtf8
	// FullName:  CharsetConvertHepler::unicodeToUtf8
	// Access:    public static 
	// Returns:   std::string 返回转换后的字符串
	// Qualifier: 将Unicode字符串转换成UTF8字符串
	// Parameter: const std::wstring & wstr Unicode字符串
	//************************************
	static std::string unicodeToUtf8(const std::wstring& wstr);

	//************************************
	// Method:    utf8ToUnicode
	// FullName:  CharsetConvertHepler::utf8ToUnicode
	// Access:    public static 
	// Returns:   std::wstring 返回转换后的字符串
	// Qualifier: 将UTF8字符串转换成Unicode字符串
	// Parameter: const std::string & str UTF8字符串
	//************************************
	static std::wstring utf8ToUnicode(const std::string& str);

	//************************************
	// Method:    unicodeToAnsi
	// FullName:  CharsetConvertHepler::unicodeToAnsi
	// Access:    public static 
	// Returns:   std::string 返回转换后的字符串
	// Qualifier: 将Unicode字符串转换成ANSI字符串
	// Parameter: const std::wstring & wstr Unicode字符串
	//************************************
	static std::string unicodeToAnsi(const std::wstring& wstr);
	
	//************************************
	// Method:    ansiToUnicode
	// FullName:  CharsetConvertHepler::ansiToUnicode
	// Access:    public static 
	// Returns:   std::wstring 返回转换后的字符串
	// Qualifier: 将ANSI字符串转换成Unicode字符串
	// Parameter: const std::string & str ANSI字符串
	//************************************
	static std::wstring ansiToUnicode(const std::string& str);

	//************************************
	// Method:    utf8ToAnsi
	// FullName:  CharsetConvertHepler::utf8ToAnsi
	// Access:    public static 
	// Returns:   std::string 返回转换后的字符串
	// Qualifier: 将UTF8字符串转换成ANSI字符串
	// Parameter: const std::string & str UTF8字符串
	//************************************
	static std::string utf8ToAnsi(const std::string& str);

	//************************************
	// Method:    ansiToUtf8
	// FullName:  CharsetConvertHepler::ansiToUtf8
	// Access:    public static 
	// Returns:   std::string
	// Qualifier: 将ANSI字符串转换成UTF8字符串
	// Parameter: const std::string & str ANSI字符串
	//************************************
	static std::string ansiToUtf8(const std::string& str);
};
#endif // _CHARSETCONVERTHEPLER_H_

```

`CharsetConvertHepler.cpp`

```c++
#include "pch.h"
#include "../include/CharsetConvertHepler.h"
#include <codecvt>
#include <iostream>
#include <wchar.h>

#ifdef LINUX
#include <stdlib.h>
#include <bits/locale_conv.h>
#endif


//转换缓冲区大小定义为1KB
#define CONVERT_BUFF_SIZE 1024

std::string CharsetConvertHepler::unicodeToUtf8(const std::wstring& wstr)
{
	std::string ret;
	try
	{
		std::wstring_convert<std::codecvt_utf8<wchar_t>> wcv;
		ret = wcv.to_bytes(wstr);
	}
	catch (const std::exception& e)
	{
		std::cerr << e.what() << std::endl;
	}
	return ret;
}

std::wstring CharsetConvertHepler::utf8ToUnicode(const std::string& str)
{
	std::wstring ret;
	try
	{
		std::wstring_convert<std::codecvt_utf8<wchar_t>> wcv;
		ret = wcv.from_bytes(str);
	}
	catch (const std::exception& e)
	{
		std::cerr << e.what() << std::endl;
	}
	return ret;
}

std::string CharsetConvertHepler::unicodeToAnsi(const std::wstring& wstr)
{
#ifdef LINUX
	setlocale(LC_CTYPE, "zh_CN.UTF-8");
	std::string ret;
	const wchar_t* src = wstr.data();
	char mbString[CONVERT_BUFF_SIZE];
	size_t size = wcstombs(mbString, src, CONVERT_BUFF_SIZE);
	if (size == static_cast<size_t>(-1))
	{
		printf("Couldn't convert string--invalid multibyte character.\n");
	}
	else
	{
		ret = mbString;
	}
	//setlocale(LC_ALL, "C");
	return ret;
#else
	setlocale(LC_ALL, "chs");
	std::string ret;
	const wchar_t* src = wstr.data();
	char mbString[CONVERT_BUFF_SIZE];
	size_t countConverted;
	errno_t err;
	//mbstate_t mbstate;
	//memset((void*)&mbstate, 0, sizeof(mbstate));
	//err = wcsrtombs_s(&countConverted, mbString, CONVERT_BUFF_SIZE, &src, _TRUNCATE, &mbstate);
	err = wcstombs_s(&countConverted, mbString, CONVERT_BUFF_SIZE, src, _TRUNCATE);
	if (err == EILSEQ)
	{
		std::cout << "字符串中存在编码错误" << std::endl;
	}
	else if (err == STRUNCATE)
	{
		std::cout << "缓存区不足，字符串过长" << std::endl;
		ret = mbString;
	}
	else
	{
		ret = mbString;
	}
	setlocale(LC_ALL, "C");
	return ret;
#endif
}

std::wstring CharsetConvertHepler::ansiToUnicode(const std::string& str)
{
#ifdef LINUX
	setlocale(LC_CTYPE, "zh_CN.UTF-8");
	std::wstring ret;
	const char* src = str.data();
	wchar_t wcstr[CONVERT_BUFF_SIZE];
	size_t size = mbstowcs(wcstr, src, CONVERT_BUFF_SIZE);
	if (size == static_cast<size_t>(-1))
	{
		printf("Couldn't convert string--invalid multibyte character.\n");
	}
	else
	{
		ret = wcstr;
	}
	//setlocale(LC_CTYPE, "C");
	return ret;
#else
	setlocale(LC_ALL, "chs");
	std::wstring ret;
	const char* src = str.data();
	wchar_t wcstr[CONVERT_BUFF_SIZE];
	size_t countConverted;
	errno_t err;
	err = mbstowcs_s(&countConverted, wcstr, CONVERT_BUFF_SIZE, src, _TRUNCATE);
	if (err == EILSEQ)
	{
		std::cout << "字符串中存在编码错误" << std::endl;
	}
	else if (err == STRUNCATE)
	{
		std::cout << "缓存区不足，字符串过长" << std::endl;
		ret = wcstr;
	}
	else
	{
		ret = wcstr;
	}
	setlocale(LC_ALL, "C");
	return ret;
#endif
}

std::string CharsetConvertHepler::utf8ToAnsi(const std::string& str)
{
	return unicodeToAnsi(utf8ToUnicode(str));
}

std::string CharsetConvertHepler::ansiToUtf8(const std::string& str)
{
	return unicodeToUtf8(ansiToUnicode(str));
}

```

# 5.雪花ID生成器

`SnowFlake.h`

```C++
#pragma once
#ifndef _SNOWFLAKE_H_
#define _SNOWFLAKE_H_
#include <stdint.h>
#include <mutex>

/**
 * 雪花ID生成工具
 * 使用示例：
 * 下面示例生成10个ID
 * SnowFlake sf(1, 1);
 * for (int i = 0; i < 10; i++)
 *	std::cout << sf.nextId() << std::endl;
 */
class SnowFlake
{
private:
	// 初始时间戳，给一个随机值
	static const uint64_t m_start_time_stamp = 1480166465631;
	// 序列号占用位数
	static const uint64_t m_sequence_bit = 12;
	// 机器ID占用位数
	static const uint64_t m_machine_bit = 5;
	// 数据标识占用位数
	static const uint64_t m_datacenter_bit = 5;

	// 获取位数的最大值
	static const uint64_t m_max_datacenter_num = -1 ^ (uint64_t(-1) << m_datacenter_bit);
	static const uint64_t m_max_machine_num = -1 ^ (uint64_t(-1) << m_machine_bit);
	static const uint64_t m_max_sequence_num = -1 ^ (uint64_t(-1) << m_sequence_bit);

	// 下标
	static const uint64_t m_machine_left = m_sequence_bit;// 机器ID向左移12位
	static const uint64_t m_datacenter_left = m_sequence_bit + m_machine_bit;// 数据标识ID向左移17位(12+5)
	static const uint64_t m_timestamp_left = m_sequence_bit + m_machine_bit + m_datacenter_bit; // 时间戳向左移22位(5+5+12)

	// 数据中心ID(0~31)
	uint64_t m_datacenterId;
	// 工作机器ID(0~31)
	uint64_t m_machineId;
	// 毫秒内序列(0~4095)
	uint64_t m_sequence;
	// 上次生成ID的时间戳
	uint64_t m_last_time_stamp;
	// 标识是否初始化完成
	bool m_is_init;
	// 线程锁
	std::mutex m_mtx;
	// 获得新的时间戳
	uint64_t getNextMill();
	// 返回以毫秒为单位的当前时间
	uint64_t getNewTimeStamp();
public:
	//************************************
	// Method:    SnowFlake
	// FullName:  SnowFlake::SnowFlake
	// Access:    public 
	// Returns:   
	// Qualifier: 构造初始化
	// Parameter: int datacenterId 数据中心ID (0~31)
	// Parameter: int machineId 工作机器ID(0~31)
	//************************************
	SnowFlake(int datacenterId, int machineId);

	//************************************
	// Method:    nextId
	// FullName:  SnowFlake::nextId
	// Access:    public 
	// Returns:   uint64_t 返回计算出来的ID，返回0表示生成ID失败
	// Qualifier: 获取下一个ID
	//************************************
	uint64_t nextId();
};


#endif // _SNOWFLAKE_H_
```

`SnowFlake.cpp`

```C++
#include "pch.h"
#include "../include/SnowFlake.h"
#include <thread>
#include <stdexcept>
#include <iostream>

uint64_t SnowFlake::getNextMill()
{
	uint64_t mill = getNewTimeStamp();
	while (mill <= m_last_time_stamp) {
		mill = getNewTimeStamp();
	}
	return mill;
}

uint64_t SnowFlake::getNewTimeStamp()
{
	auto t = std::chrono::time_point_cast<std::chrono::milliseconds>(std::chrono::high_resolution_clock::now());
	return t.time_since_epoch().count();
}

SnowFlake::SnowFlake(int datacenterId, int machineId)
{
	m_datacenterId = 0L;
	m_machineId = 0L;
	m_sequence = 0L;
	m_last_time_stamp = 0L;
	m_is_init = false;
	if ((uint64_t)datacenterId > m_max_datacenter_num || (uint64_t)datacenterId < 0) {
		std::cerr << "datacenterId can't be greater than max_datacenter_num_ or less than 0" << std::endl;
		return;
	}
	if ((uint64_t)machineId > m_max_machine_num || (uint64_t)machineId < 0) {
		std::cerr << "machineId can't be greater than max_machine_num_or less than 0" << std::endl;
		return;
	}
	m_datacenterId = datacenterId;
	m_machineId = machineId;
	m_is_init = true;
}

uint64_t SnowFlake::nextId()
{
	// 构造初始化错误，不执行后续逻辑
	if (!m_is_init) return 0;

	std::unique_lock<std::mutex> lock(m_mtx);
	uint64_t curTimeStamp = getNewTimeStamp();
	// 如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
	if (curTimeStamp < m_last_time_stamp) {
		std::cerr << "clock moved backwards. refusing to generate id" << std::endl;
		return 0;
	}
	// 如果是同一时间生成的，则进行毫秒内序列
	if (curTimeStamp == m_last_time_stamp) {
		m_sequence = (m_sequence + 1) & m_max_sequence_num;
		// 毫秒内序列溢出
		if (m_sequence == 0) {
			// 获取下一个毫秒时间戳
			curTimeStamp = getNextMill();
		}
	}
	// 时间戳改变，毫秒内序列重置
	else
	{
		m_sequence = 0;
	}
	// 更新上次生成ID的时间戳
	m_last_time_stamp = curTimeStamp;
	// 移位并通过或运算拼到一起组成64位的ID
	return (curTimeStamp - m_start_time_stamp) << m_timestamp_left
		| m_datacenterId << m_datacenter_left
		| m_machineId << m_machine_left
		| m_sequence;
}
```

