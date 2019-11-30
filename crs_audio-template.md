# 语音报告模板



```php+HTML
{patients_name}, {patients_title}，您好！

您的云健康监测报告内容如下：

您现在的健康状况需要引起重视，在本次检测中存在风险预警的疾病主要有：

{diseases_list}



风险疾病解读如下：

{disease_name}的疾病风险预警值为：{level_string}。

{disease_descriptions}

{disease_name}属于{biosystem_name}疾病。

针对这些问题，云健康中心建议您：

{biosystem_basic_advice}

此外，我们还推荐：

{biosystem_special_advice}



如果您对该检测报告有疑问，请您联系您的健康管理师，也可以扫描报告末页的二维码，添加微信关注我方公众号，或致电我中心，联系电话为：05332791000

与将康中心祝您鑫福安康！
```



> - MP3文件名：直接从`patients` 表读取`p_audio`值，并对字符串做切割处理得到类似"CR01201911012324.mp3" 字样的文件名。
> - `{patients_name}`: 直接从`patients` 表中读取`p_name`。
> - `{patients_title}`: 判断从`patients`表中读取的`p_gender`值
>   - `p_gender == '男' ==> {patients_title} = '先生'` 
>   - `p_gender == '女' ==> {patients_title} = '女士'` 
> - `{disease_list}`: 从`results`表中根据`patients_id` 和`r_date` 获得检测出的疾病列表，并按照D值(`d_spectrum`)从小到大排列
> - `{disease_name}`: 在上面`{disease_list}` 中的疾病，在`diseases` 表中查找是否存在。
>   - 如果在`diseases` 表中查询到：看看说属的系统，一个系统中只读一个疾病D值最小的疾病。
>   - 如果没有，直接排除`{disease_list}`中的疾病
> - `{level_string}`: 通过判断从`results` 中获取的`r_level` 数值：
>   - `r_level == 1  ==> {level_string} = '高'` 
>   - `r_level == 2  ==> {level_string} = '中'`
> - `{disease_descriptions}` 和 `{biosystem_name}`: 直接从`disease` 表根据疾病名获取
> - `{biosystem_basic_advice}` 和 `{biosystem_special_advice}`: 直接从`biosystems`表中获取