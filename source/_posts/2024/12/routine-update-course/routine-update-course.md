---
title: 更新网站上课表的步骤
date: 2024-12-15
permalink: 2024/12/routine-update-course.md
categories: 个人工作
---

## 更新时间

课表更新的时间有三个阶段：

* 上学期 `16` 周左右的第一轮选课前夕；
* 下学期开学后第四轮选课结束，教务处发布某些选修课调整关停的通知后；
* 下学期期中退课结束后。

## 更新流程与方法

### 在个人电脑上

> **记得备份！记得备份，记得备份！**

#### Excel 表格

访问 `1.tongji.edu.cn`，切换到全校课表，把对应学期的全校课表导出为 `Excel` 表格；

![步骤一](https://static.xialing.icu/img/202412150945515.webp)

#### 导出为 .csv

把 `Excel` 表格导出为 `.csv` 文件；

方法：`文件`->`另存为`->`选择位置`->`在下拉菜单选择(.csv)`->`保存`

![步骤二的部分展示](https://static.xialing.icu/img/202412150949919.webp)

这两个有什么区别吗？如果选择**不含有** `UTF-8` 的（靠下面的），编码会变成 `GB2312`，在进行 `pandas` 处理的时候会有问题，提示无法读取对应编码（因为 `pandas` 默认读 `UTF-8`），然而如果在 `python` 程序中指定 `coding` 似乎也无法解决问题，所以这时，可以通过 `VSCode` 打开 `.csv` 文件，先选择 `按编码打开`，`GB2312`；再选择 `按编码保存`，`UTF-8 with BOM`，解决！至于说没有 `BOM` 的单纯 `UTF-8` 怎样，以后可以试试..

#### 进行数据处理

```python
import os
import pandas as pd
import time


# 获取当前文件夹中所有 .csv 文件
csv_files = [f for f in os.listdir('./ori_csv') if f.endswith('.csv') and '学年' in f]
print(csv_files) # 打印读取到的文件

# 获取当前文件夹中所有已处理的 .csv 文件
edited_files = [f for f in os.listdir('.') if f.endswith('_edited.csv')]
print(edited_files) # 打印读取到的文件

# 过滤掉已经处理过的文件
csv_files = [f for f in csv_files if f.replace('.csv', '_edited.csv') not in edited_files]
print(csv_files) # 打印读取到的文件

# 用于存储所有修改后的 DataFrame
processed_dfs = []

for file in csv_files:
    # 从文件名提取学期信息，例如 "2007-2008学年第1学期"
    semester = file.split('工作安排表')[0]

    # 读取 CSV 文件
    ori_path = './ori_csv/'
    df = pd.read_csv(ori_path + file, skiprows=2) # 忽略前两行，一行是标题（某某学期工作安排表），另一行是 ,,,,,,,,,

    # print(df)

    # 删除“负责人”列（如果存在）
    if "负责人" in df.columns:
        df = df.drop(columns=["负责人"])
    
    # 添加“开课学院”列并初始化为空字符串
    df['开课学院'] = ''

    # 处理“开课学院”的逻辑
    current_department = ''
    skip_next = False  # 用于控制跳过下一行的标志
    
    for i, row in df.iterrows():
        # print(row)
        # time.sleep(1)
        if skip_next:
            # 如果需要跳过当前行，则重置标志并继续下一次循环
            skip_next = False
            continue

        # 如果第1列是 NaN 且第0列是字符串，则更新当前的学院名称
        if pd.isna(row[1]) and isinstance(row[0], str):
            current_department = row[0].strip()
            skip_next = True  # 设置标志跳过下一行
            
        elif pd.isna(row[0]) and pd.isna(row[1]):
            current_department = ''
            skip_next = True  # 设置标志跳过下一行
            
        else:
            # 否则，设置“开课学院”列的值
            df.at[i, '开课学院'] = current_department

    # print(df)
    
    # 删除部门名行（第1列为 NaN 的行）
    df = df.dropna(subset=[df.columns[1]])

    # print(df)
    # 添加“学期”列
    df.insert(0, '学期', semester)
    # print(df)

    df = df.applymap(lambda x: x.rstrip() if isinstance(x, str) else x) # 去除空白符
    
    # 重置索引
    df = df.reset_index(drop=True)

    # 将修改后的 DataFrame 添加到列表
    processed_dfs.append(df)

    # 保存修改后的文件为 "xxx_edited.csv"
    new_filename = file.replace('.csv', '_edited.csv')
    df.to_csv(new_filename, index=False, header=True)

if processed_dfs:
    # 合并所有修改后的 DataFrame
    merged_df = pd.concat(processed_dfs, ignore_index=True)

    # 检查 merged_schedule.csv 是否存在
    if os.path.exists('merged_schedule.csv'):
        # 如果存在，读取现有的 merged_schedule.csv
        existing_df = pd.read_csv('merged_schedule.csv')
        # 将新的数据合并到现有的 DataFrame
        merged_df = pd.concat([existing_df, merged_df], ignore_index=True)

    # 保存合并后的文件
    merged_df.to_csv('merged_schedule.csv', index=False, header=True)

    print("处理完成，所有文件已保存并合并为 merged_schedule.csv")
else:
    print("没有找到需要处理的文件或所有文件已处理过。")
```

现在的处理逻辑是：

1. 先查找有没有新的课程表添加进来；
2. 对新添加的课表进行处理，生成 `*_edited.csv` 文件；
3. 读取原来的 `*merged.csv` 文件到内存中，把新课表 `append` 到后面。
4. 保存为新的 `*merged.csv` 文件。

所以注意，在进行新学期的添加前，记得备份！记得备份，记得备份！否则就被覆盖了。

#### 导入到数据库

其实你说，真的有必要生成一份 `*merged.csv` 文件吗？没必要。只需要把新学期的课程 `LOAD` 到课表里就行了。不过为了保险，还是存一份原始 `.csv` 文件吧，方便迁移？或许。

导入到数据库，除非哪天数据库不小心被我删了，否则**只需要插入更新的一个学期的内容就可以了**，当然现在还没有经历过第一部分说的两个步骤，后续如果需要修改某学期的内容，需要先把对应学期的内容删除掉（否则会有主键重复），到时候再更新这篇文档。

导入到数据库的原始代码如下：

```sql
LOAD DATA INFILE '/path/to/24252.csv' -- 原始文件路径
INTO TABLE course_all                 -- 插入表格
FIELDS TERMINATED BY ','              -- 逗号分隔
ENCLOSED BY '"'                       -- "" 包裹
LINES TERMINATED BY '\n'              -- \n 结束一行
IGNORE 1 LINES;                       -- 忽略第一行（表头）
```

#### 把表格导出为 `.sql` 文件，迁移到服务器

![3-1](https://static.xialing.icu/img/202412151008920.webp)

![3-2](https://static.xialing.icu/img/202412151011453.webp)

这样就得到了 `.sql` 文件。

### 在服务器上

#### 使用 `XSHELL` 连接到服务器

`oop` 已经使用过很多次了，不赘述。

#### 把 `.sql` 文件通过 `XFTP` 上传到对应文件夹

和本地的配置不同，服务器上 `app.py` 的配置体如下：

```python
db_config = {
    'host': 'localhost',      # 数据库主机地址，通常是 'localhost' 或远程服务器 IP
    'user': 'zhangsan',  # 连接数据库的用户名
    'password': '******', # password
    'database': 'ABCDE',    # 数据库名
    'charset': 'utf8mb4',     # 字符集设置，保证支持中文等多语言字符
}
```

这样可以确保安全，使得后端访问数据库的时候只能读，不能写。这里只是说一句，重点在于，把 `.sql` 导入到数据库后，对应的 `DATABASE` 名和 `TABLE` 名要对应，不然查找不到东西！

如果存在某个 `<database>`，直接运行：

```bash
sudo mysql your_database_name < /path/to/your_file.sql
```

即可。

换句话说，就是不需要进入到 `mysql` 进程当中。如果不小心进入了，输入 `EXIT;` 来退出吧！

如果对应的 `<database>` 不存在，需要手动创建。这时候需要进入 `mysql` 进程，进行如下操作：

```sql
CREATE <database>;
```

语法应该没错。创建完就 `OK` 了，可以导入。不用新建表格，设计表的结构，因为直接导入来的 `.sql` 文件包含了这些信息。（当然到目前为止，这部分内容是我凭空揣测的，等遇到实际应用时再修正）。

#### 重启服务

有一个 `rfs` 别名，可能是 `Restart Flask Service` 的意思吧，忘了具体含义了。别名对应关系是：

```bash
rfs='sudo systemctl restart nginx; sudo systemctl restart flask_app.service'
```

那这个 `flask_app.service` 是啥呢？它是一个系统服务，在我的服务器上存放在 `/etc/systemd/system` 下，内容是：

```ini
[Unit]
Description=Gunicorn instance to serve Flask application
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/jkd-web/flask-backend/
Environment="PATH=/home/ubuntu/jkd-web/flask-backend/venv/bin" # 当时找这个环境费了我不少时间
ExecStart=/home/ubuntu/jkd-web/flask-backend/venv/bin/gunicorn -w 4 -b 127.0.0.1:8000 app:app
[Install]
WantedBy=multi-user.target
```

这段代码完全是 `AI` 生成的，现在我几乎看不懂了。以后写大作业的时候一定要留一份文档，给自己看也好。

### 结束..之前

执行到此，应该可以正常在 `tongji.xialing.icu` 中查询到最新的课程了！这次更新也说明了：我的网站可以自己定位到最近的学期进行选择，很方便。

#### 更新日志

1. 在主页更新本次更新的内容和时间；
2. 在 `footer` 更新本次更新的时间。
