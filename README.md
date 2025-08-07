# DocumentDB Serverless 评估工具

## 🚀 工具简介

DocumentDB Serverless 评估工具是一个自动化的成本和性能分析工具，帮助客户评估从 Amazon DocumentDB 预置实例（On-Demand）迁移到 DocumentDB Serverless 的可行性。

### 核心功能
- **自动化数据采集**: 自动收集客户数据库运行环境的 CPU 使用率基础数据
- **实例价格分析**: 获取最新的 DocumentDB 实例和 DCU 价格信息
- **成本效益评估**: 从成本角度比较 On-Demand 实例与 Serverless 的月度开销
- **性能影响分析**: 以 CPU 利用率为主要判断依据，评估迁移的性能影响
- **可视化报告**: 生成包含图表和详细分析的 Excel 报告

## 📋 评估维度

### 成本维度
- **On-Demand 成本**: 基于当前实例类型和数量的月度成本
- **Serverless 成本**: 基于 CPU 使用率估算的 DCU 消费成本
- **成本节省分析**: 计算潜在的成本节省金额和百分比
- **ROI 分析**: 提供迁移建议和投资回报分析

### 性能维度
- **CPU 使用率分析**: 分析平均、最小、最大 CPU 使用率
- **资源利用率分布**: 生成 CPU 使用率分布饼图
- **容量规划**: 基于历史数据预测 Serverless 容量需求

## 🛠️ 环境安装 (Amazon Linux)

### 1. 系统更新
```bash
# 更新系统包
sudo yum update -y
```

### 2. Python 环境准备
```bash
# Amazon Linux 2 通常已预装 Python 3
python3 --version

# 如果没有 Python 3，安装它
sudo yum install python3 -y

# 安装 pip
sudo yum install python3-pip -y
```

### 3. 系统依赖安装
```bash
# 安装图形库依赖 (matplotlib 需要)
sudo yum install -y gcc python3-devel
sudo yum install -y freetype-devel libpng-devel
```

### 4. Python 库安装
```bash
# 安装必需的 Python 库 (全局安装)
sudo pip3 install boto3
sudo pip3 install pandas
sudo pip3 install openpyxl
sudo pip3 install matplotlib
sudo pip3 install numpy

# 一次性安装所有依赖
sudo pip3 install boto3 pandas openpyxl matplotlib numpy
```

### 5. AWS CLI 安装和配置
```bash
# 安装 AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# 配置 AWS 凭证
aws configure
```

### 6. 验证安装
```bash
# 验证 Python 库安装
python3 -c "import boto3, pandas, openpyxl, matplotlib, numpy; print('所有依赖库安装成功')"

# 验证 AWS 配置
aws sts get-caller-identity
```

## 🔧 工具使用

### 1. 下载工具
```bash
# 克隆仓库
git clone https://github.com/bingbingliu18/docDB-Serverless.git
cd docDB-Serverless

# 或直接下载主文件
wget https://raw.githubusercontent.com/bingbingliu18/docDB-Serverless/main/DocumentDB_serverless_cost_comparison.py
```

### 2. 运行工具
```bash
# 确保文件有执行权限
chmod +x DocumentDB_serverless_cost_comparison.py

# 运行评估工具
python3 DocumentDB_serverless_cost_comparison.py
```

### 3. 区域选择
工具启动后会显示支持的 AWS 区域列表：
```
Please select a region by entering the corresponding number:
1. us-east-1
2. ap-northeast-1
3. us-east-2
...
Enter your choice (1-18): 
```

### 4. 处理流程
1. **集群发现**: 自动发现指定区域的所有 DocumentDB 集群
2. **数据采集**: 收集过去30天的 CPU 使用率监控数据
3. **价格查询**: 获取最新的实例和 DCU 价格信息
4. **成本计算**: 计算 On-Demand 和 Serverless 的月度成本
5. **报告生成**: 生成详细的分析报告和可视化图表

## 📊 报告输出

### 输出文件
- **Excel 报告**: `docdb_cost_comparison_report.xlsx`
- **可视化图表**: 
  - `docdb_cost_comparison_chart.jpg` - 成本比较柱状图
  - `docdb_cpu_usage_pie.jpg` - CPU 使用率分布饼图
- **日志文件**: `docdb_cost_comparison_log_YYYYMMDD_HHMMSS.log`

### 报告内容

#### Summary 工作表
- CPU 使用率分布饼图
- 成本比较柱状图（显示前8个集群）
- 总体成本节省摘要

#### Detail 工作表
| 字段 | 描述 |
|------|------|
| account_id | AWS 账户 ID |
| region | AWS 区域 |
| cluster_id | DocumentDB 集群 ID |
| engine | 数据库引擎类型 |
| engine_version | 引擎版本 |
| instance_type | 实例类型 |
| vcpu | 虚拟 CPU 数量 |
| CPU Avg Util% | 平均 CPU 使用率 |
| CPU Min/Max Util% | 最小/最大 CPU 使用率 |
| OnDemand Monthly Cost | On-Demand 月度成本 |
| Serverless Monthly Cost | Serverless 月度成本 |
| Cost Savings | 成本节省金额 |
| Savings % | 节省百分比 |
| Recommendation | 迁移建议 |

#### Cost Summary 工作表
- 简化的成本比较摘要
- 推荐方案统计
- 总体 ROI 分析

### 示例输出
```
=== DocumentDB Serverless 评估结果 ===
总On-Demand月成本: $2,456.78
总Serverless月成本: $1,834.56
总节省金额: $622.22
总节省百分比: 25.3%

=== 迁移建议统计 ===
推荐迁移到Serverless的集群: 7
推荐保持On-Demand的集群: 3
总集群数: 10
```

## 🔍 技术实现

### 核心算法
```python
# DCU 使用量估算
estimated_dcu_per_instance = vcpu * (avg_cpu_util / 100.0)
min_dcu_per_instance = max(0.5, estimated_dcu_per_instance)  # 最小 0.5 DCU

# Serverless 月度成本计算
total_dcu_hours = min_dcu_per_instance * instance_count * 730
monthly_serverless_cost = total_dcu_hours * dcu_price_per_hour
```

### 关键特性
- **并发处理**: 使用 ThreadPoolExecutor 提高数据采集效率
- **智能缓存**: 避免重复查询相同的存储和价格数据
- **错误恢复**: 完善的异常处理和重试机制
- **进度跟踪**: 实时显示处理进度和状态

## 🔐 权限要求

### 必需的 IAM 权限
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "docdb:DescribeDBClusters",
                "docdb:DescribeDBInstances",
                "cloudwatch:GetMetricData",
                "pricing:GetProducts"
            ],
            "Resource": "*"
        }
    ]
}
```

### EC2 实例角色配置 (推荐)
```bash
# 如果在 EC2 上运行，推荐使用 IAM 角色而不是访问密钥
# 创建角色并附加上述权限策略，然后将角色附加到 EC2 实例
```

## 🌍 支持的区域

工具支持以下 AWS 区域：
- **美国**: us-east-1, us-east-2, us-west-1, us-west-2
- **亚太**: ap-northeast-1, ap-northeast-2, ap-southeast-1, ap-southeast-2, ap-south-1, ap-east-1
- **欧洲**: eu-central-1, eu-west-1, eu-west-2, eu-west-3, eu-north-1
- **其他**: ca-central-1, me-south-1, sa-east-1

## ⚠️ 注意事项

### 数据要求
- 集群需要有至少 **30天** 的 CloudWatch 监控数据
- 确保 CloudWatch 详细监控已启用

### 成本估算精度
- Serverless 成本基于 CPU 使用率估算，实际成本可能有 ±10-20% 的差异
- 建议结合实际业务模式进行最终决策

### Amazon Linux 特定注意事项
- 确保系统有足够的磁盘空间存储报告文件
- 如果遇到权限问题，可能需要使用 `sudo` 运行某些命令
- 建议定期更新系统包以确保兼容性

## 🔧 故障排除

### 常见问题

#### 1. 导入错误
```bash
# 如果遇到 "No module named 'xxx'" 错误
sudo pip3 install --upgrade pip
sudo pip3 install 模块名
```

#### 2. matplotlib 显示问题
```bash
# 如果遇到图形显示问题，设置后端
export MPLBACKEND=Agg
python3 DocumentDB_serverless_cost_comparison.py
```

#### 3. 权限问题
```bash
# 确保 AWS 凭证正确配置
aws configure list
aws sts get-caller-identity
```

#### 4. 网络连接问题
```bash
# 检查网络连接
curl -I https://aws.amazon.com
```

## 📞 支持

如有问题或建议，请：
1. 提交 [GitHub Issue](https://github.com/bingbingliu18/docDB-Serverless/issues)
2. 查看日志文件获取详细错误信息
3. 确认 AWS 权限和网络连接正常

## 📄 快速开始示例

```bash
# 完整的安装和运行流程
sudo yum update -y
sudo yum install python3 python3-pip gcc python3-devel freetype-devel libpng-devel -y
sudo pip3 install boto3 pandas openpyxl matplotlib numpy
aws configure
git clone https://github.com/bingbingliu18/docDB-Serverless.git
cd docDB-Serverless
python3 DocumentDB_serverless_cost_comparison.py
```

---

**免责声明**: 此工具提供的成本估算仅供参考，实际成本可能因具体使用模式而异。建议在生产环境迁移前进行小规模测试验证。
