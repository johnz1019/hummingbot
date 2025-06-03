# Executor Activation Bounds 日志增强总结

本文档总结了为 hummingbot 策略中 executor_activation_bounds 相关逻辑添加的详细日志记录。

## 修改的文件和方法

### 1. DCA Executor (`hummingbot/strategy_v2/executors/dca_executor/dca_executor.py`)

#### `control_open_order_process()` 方法
- **添加的日志**: 
  - 成功创建订单时记录 INFO 日志
  - 由于激活边界或过期无法创建订单时记录 WARNING 日志
- **日志内容**:
  - 执行器 ID
  - DCA 层级
  - 订单价格和当前价格
  - 失败原因（激活边界 vs 过期）

#### `_is_within_activation_bounds()` 方法
- **MAKER 模式日志**:
  - BUY: 记录订单价格、当前价格、阈值价格、激活边界值、检查结果
  - SELL: 记录订单价格、当前价格、阈值价格、激活边界值、检查结果
  - 无激活边界时记录允许执行的信息

- **TAKER 模式日志**:
  - BUY: 记录订单价格、当前价格、最小/最大价格范围、激活边界、检查结果
  - SELL: 记录订单价格、当前价格、最小/最大价格范围、激活边界、检查结果

### 2. Position Executor (`hummingbot/strategy_v2/executors/position_executor/position_executor.py`)

#### `control_open_order()` 方法
- **添加的日志**:
  - 激活边界检查通过时记录 INFO 日志
  - 激活边界检查失败时记录 WARNING 日志
  - 订单不再满足激活边界时记录取消日志

#### `_is_within_activation_bounds()` 方法
- **LIMIT 订单日志**:
  - BUY: 记录订单价格、中间价、阈值价格、激活边界、检查结果
  - SELL: 记录订单价格、中间价、阈值价格、激活边界、检查结果

- **MARKET 订单日志**:
  - BUY: 记录订单价格、中间价、最小/最大价格范围、激活边界、检查结果
  - SELL: 记录订单价格、中间价、最小/最大价格范围、激活边界、检查结果

- **无激活边界**: 记录允许执行的信息

### 3. Grid Executor (`hummingbot/strategy_v2/executors/grid_executor/grid_executor.py`)

#### `_filter_levels_by_activation_bounds()` 方法
- **添加的日志**:
  - 记录网格过滤的详细信息
  - BUY/SELL 分别记录中间价、激活边界、阈值价格
  - 记录总层级数和过滤后的层级数
  - DEBUG 级别记录每个层级的价格和是否允许执行

## 日志级别说明

### INFO 级别
- 激活边界检查的详细计算过程
- 成功创建订单的确认信息
- 正常的过滤和检查结果

### WARNING 级别
- 由于激活边界限制无法创建订单
- 订单被取消（不再满足激活边界）
- 由于过期无法创建订单

### DEBUG 级别
- 网格执行器中每个层级的详细检查结果

## 使用这些日志进行调试

### 1. 检查DCA为什么不下单
查找类似日志：
```
DCA Executor ID: xxx - Level 0 not created due to activation bounds!
DCA Executor ID: xxx - MAKER BUY activation bounds check: Order price: 0.06105, Current price: 0.06105, Threshold price: 0.060439, Activation bound: 0.001, Within bounds: True
```

### 2. 检查Position Executor问题
查找类似日志：
```
Position Executor ID: xxx - Activation bounds check failed, not placing open order!
Position Executor ID: xxx - LIMIT BUY activation bounds check: Within bounds: False
```

### 3. 检查Grid Executor过滤
查找类似日志：
```
Grid Executor ID: xxx - BUY activation bounds filter: Total levels: 10, Filtered levels: 3
```

## 配置建议

根据日志结果，你可以调整激活边界：

### 放宽限制
```yaml
executor_activation_bounds: [0.02]  # 从0.001改为0.02 (2%)
```

### 关闭限制
```yaml
executor_activation_bounds: null  # 完全关闭
```

### 不同模式的典型设置
- **保守**: `[0.001]` (0.1%)
- **平衡**: `[0.01]` (1%)
- **激进**: `[0.05]` (5%)
- **无限制**: `null` 