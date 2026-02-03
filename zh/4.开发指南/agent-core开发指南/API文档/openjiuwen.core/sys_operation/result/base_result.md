# openjiuwen.core.sys_operation.result.base_result

`openjiuwen.core.sys_operation.result.base_result` 模块定义了所有操作结果的基类。

## class BaseResult

```python
class BaseResult(BaseModel, Generic[T], ABC):
    code: int = Field(..., description="Status code: 0 = success, non-0 = failure")
    message: str = Field(..., description="Message details")
    data: Optional[T] = Field(None, description="Business data (returned only on success)")
```

所有操作结果的通用泛型基类。

### 字段

- `code: int`：
  - 状态码。
  - `0` 表示成功。
  - 非 `0` 表示失败。
- `message: str`：
  - 消息详情。
- `data: Optional[T]`：
  - 业务数据载荷（仅在成功时返回）。
  - 默认为 `None`。
