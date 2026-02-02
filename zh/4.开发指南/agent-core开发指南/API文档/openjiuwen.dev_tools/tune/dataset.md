# openjiuwen.dev_tools.tune.dataset

## class openjiuwen.dev_tools.tune.dataset.case_loader.CaseLoader

`CaseLoader` 用于管理调优/评测数据集（持有 `Case` 列表并提供 shuffle/split 等操作）。

它既可以作为 `Trainer.train/evaluate` 的输入，也可以在自定义训练流程中被迭代消费。

## 初始化

```python
CaseLoader(cases: List[Case])
```

**参数**：

- **cases**：样本列表。
  - 会在初始化时为每个样本分配稳定的 `case_id`（形如 `case_0`、`case_1`）。
  - 若你希望保留原始 uuid，请避免再调用 `CaseLoader._assign_case_id()`（内部实现细节）。

## 常用方法

### get_cases

```python
get_cases(self) -> List[Case]
```

返回内部持有的样本列表（按当前顺序）。

### shuffle

```python
shuffle(self, random_seed: int = 0)
```

原地打乱样本顺序，并重新分配 `case_id` 以反映新顺序。

**参数**：

- **random_seed**：随机种子。
  - 用于保证实验可复现。
  - 同一 seed + 同一初始样本顺序将得到相同 shuffle 结果。

### split

```python
split(self, ratio: float = 0.5) -> Tuple[CaseLoader, CaseLoader]
```

按比例拆分数据集为两部分（常用于 train/val 切分）。

**参数**：

- **ratio**：切分比例（0~1）。
  - 表示第一部分占比（默认 0.5）。
  - 若 ratio 越界，会记录错误并回退到默认 0.5（避免直接抛异常中断流程）。

**返回**：

- `(train_loader, val_loader)`：两个新的 `CaseLoader` 实例（内部会 deepcopy 样本以避免互相影响）。

