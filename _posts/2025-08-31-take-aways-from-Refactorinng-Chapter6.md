---
layout: article
title: 《重构》第六章学习总结：从代码演进中理解重构的艺术
description: 深度解析《重构》第六章的11种核心重构手法，从提取函数到拆分阶段，通过实际代码示例展示如何在不改变外部行为的前提下持续提升代码质量。包含重构避坑指南和实践检查清单，帮助开发者建立小步快跑、测试驱动的重构思维。
tags: [reading, reference]
lang: zh
---

> **“重构不是重写，而是在不改变外部行为的前提下，持续提升代码的可读性与可维护性。”**  
> —— 基于《重构：改善既有代码的设计（第二版）》第六章实践练习的深度总结

本篇总结基于对《重构》第六章练习代码[^1]（`src/Chapter6`）及其测试（`tests/Chapter6`）的分析，结合提交历史，系统梳理了本章所涵盖的11种核心重构手法。整个过程体现了**小步快跑、测试驱动、逐步演进**的重构哲学。

## 一、重构的核心目标

重构的本质不是“把代码改写成多么复杂或者酷炫的样子”，而是：

- **提升可读性**：让代码“自解释”，减少认知负担  
- **增强可维护性**：为未来修改提供安全、清晰的路径  
- **降低耦合度**：通过封装与职责分离，减少“牵一发而动全身” 的情况
- **建立安全网**：以测试为保障，确保行为不变  

重构不是一次性大手术，而是**持续的小步优化**，每一步都应有测试护航。

## 二、11种重构手法详解

### 1. **提取函数（Extract Function）**

将复杂的逻辑块拆分为命名清晰、职责单一的函数。

```python
# 重构前
total = sum([score for score in data['scores']])
avg = total / len(data['scores'])
status = "Pass" if avg >= 60 else "Fail"
f"Total: {total}, Avg: {avg}, Status: {status}"

# 重构后
def sum_total_score(scores): ...
def cal_average(total, count): ...
def determine_status(avg): ...
def generate_report(data): ...
```

**好处**：
- 函数名即文档，提升可读性  
- 便于独立测试与复用  
- 降低主函数复杂度  

**实践要点**：提取后及时**重命名**，确保函数名准确表达意图。

### 2. **内联函数（Inline Function）**

当函数体足够简单且仅被调用一次时，可考虑内联以减少间接调用。

```python
# 内联前
def get_price():
    return 100

price = get_price()

# 内联后
price = 100
```

**适用场景**：
- 函数名未提供额外信息  
- 调用仅一处，且逻辑简单  

**避免滥用，可读性优先**：不要为了“减少函数数量或者代码长度”而内联，有很多情况下内联后可能会让代码变得特别晦涩，要观察很久才能看出其中逻辑。例如下面就是一些反例：

```python
# 内联前
def is_eligible_for_discount(order_amount):
	return order_amount > 1000

if is_eligible_for_discount(total_price):
	apply_discount()


# 内联后 - 业务规则失去语义表达
if total_price > 1000:
	apply_discount()
```

```python
# 内联前
def calculate_tax(subtotal):
	base_tax = subtotal * 0.1
	return base_tax + 5 if base_tax > 50 else base_tax

tax = calculate_tax(subtotal)

# 内联后 - 计算细节污染调用处
tax = subtotal * 0.1
tax = tax + 5 if tax > 50 else tax # 内联后需额外注释说明逻辑
```

```python
# 内联之前：函数封装了复杂的业务规则，内联后难以理解
def is_eligible_for_premium_membership(customer):
    return (  
    customer.age >= 18  
    and customer.account_age_in_years >= 2  
    and (  
        customer.annual_spending > 1000  
        or (customer.annual_spending > 500 and customer.referral_count >= 3)  
    )  
    and not customer.has_outstanding_complaints  
)

def calculate_membership_level(customer):
    if is_eligible_for_premium_membership(customer):
        return "Premium"
    elif customer.annual_spending > 300:
        return "Standard"
    else:
        return "Basic"


# 内联之后
def calculate_membership_level(customer):
    if (  
	    customer.age >= 18  
	    and customer.account_age_in_years >= 2  
	    and (  
	        customer.annual_spending > 1000  
	        or (customer.annual_spending > 500 and customer.referral_count >= 3)  
	    )  
	    and not customer.has_outstanding_complaints  
	):  
	    return "Premium"
    elif customer.annual_spending > 300:
        return "Standard"
    else:
        return "Basic"
```

### 3. **提取变量（Extract Variable）**

为复杂表达式引入中间变量，提升表达式的可读性。

```python
# 重构前
if (price * quantity * (1 + tax_rate)) > 1000:
    apply_discount()

# 重构后
total_with_tax = price * quantity * (1 + tax_rate)
if total_with_tax > 1000:
    apply_discount()
```

**核心价值**：**命名即解释**，降低理解成本。

**命名建议**：使用 `total_with_tax`、`is_eligible` 等**语义明确**的名称。

### 4. **内联变量（Inline Variable）** `m04`

与提取变量相反，当临时变量无实际语义价值时，可直接内联。

```python
# 内联前
temp = calculate_score()
return temp * 2

# 内联后
return calculate_score() * 2
```

**判断标准**：
- 变量名是否比表达式更清晰？  
- 是否有助于调试或文档化？  

若答案为“否”，则可内联。

### 5. **修改函数声明（Change Function Declaration）**

安全地修改函数签名：**重命名、增删参数、调整参数顺序**。

**安全策略**：
1. 先保留旧函数作为**包装器（wrapper）**，提取其中的代码到新的函数
2. 逐步迁移调用方
3. 最终移除旧接口

**关键实践**：**渐进式修改**，避免一次性大规模变更导致测试失败。

### 6. **封装变量（Encapsulate Variable）**

将模块级的可变状态（如全局字典）封装，避免外部随意修改。

```python
# 重构前
config = {"debug": True, "timeout": 30}

# 重构后
_config = {"debug": True, "timeout": 30}
def get_config():
    return MappingProxyType(_config)  # 只读视图
```

**进阶技巧**：
- 使用 `MappingProxyType` 提供**零拷贝的只读视图**
- 综合考虑性能开销、结构层级来选择 `copy`, `deep_copy` 和 `MappingProxyType`
- 避免 `dict.copy()` 带来的性能开销与潜在误用

**核心思想**：**封装不仅是加 getter/setter，更是控制所有权与访问语义**。

### 7. **重命名变量（Rename Variable）**

通过命名揭示意图，是成本最低但收益最高的重构之一。

```python
# 重构前
flag = user.age >= 18
if flag: ...

# 重构后
is_adult = user.age >= 18
if is_adult: ...
```

✅ **命名原则**：
- 使用 `is_`、`has_`、`can_` 等前缀表达布尔含义  
- 避免 `temp`、`data`、`result` 等模糊名称  

**测试同步**：重命名后，测试中的断言也应同步更新，保持一致性。

### 8. **引入参数对象（Introduce Parameter Object）**

将一组相关的参数封装为对象，提升可维护性。

```python
# 重构前
def check_vital_sigs(systolic, diastolic, pulse):
    ...

# 重构后
class BloodPressure:
    def __init__(self, systolic, diastolic):
        self.systolic = systolic
        self.diastolic = diastolic
    def is_normal(self): ...

def check_vital_sigs(bp: BloodPressure, pulse):
    ...
```

**优势**：
- 减少参数列表长度  
- 集中领域逻辑（如血压分类）  
- 便于扩展（如增加单位、校验）  

**适用场景**：多个函数共享相同参数组合时。

### 9. **将函数组合成类（Combine Functions into Class）**

将一组相关函数与数据封装为类，提升内聚性。

```python
# 重构前
def calculate_discount(order): ...
def apply_discount(order, discount): ...

# 重构后
class OrderDiscounts:
    def __init__(self, order): ...
    def _calculate_discount(self): ...
    def _apply_discount(self): ...
    def apply(self): ...
```

**演进路径**：
- 先组合为类  
- 后续可引入**多态**替代条件判断（如不同会员等级折扣）  

**私有方法命名**：使用 `_` 前缀（如 `_calculate_discount`）明确内部实现。

### 10. **将函数组合成变换（Combine Functions into Transform）**

将多个数据增强函数合并为一个**纯函数**，输出富化后的数据。

```python
def enrich_order_data(order):
    order = add_tax_info(order)
    order = add_discount_info(order)
    order = add_shipping_status(order)
    return order
```

**特点**：
- 不修改原数据（**不可变性**）  
- 易于测试与组合  
- 适合数据流水线场景  

**测试重点**：确保原始字段未被意外修改。

### 11. **拆分阶段（Split Phase）**

将长函数按逻辑阶段拆分为多个步骤，提升可读性与可测试性。

```python
def process_order(order):
    # Phase 1: 数据解析
    parsed = parse_order_data(order)
    # Phase 2: 业务计算
    discount = calculate_discount(parsed)
    total = calculate_total(parsed, discount)
    # Phase 3: 结果格式化
    return format_result(total)
```

**实践建议**：
- 每个阶段提取为独立函数  
- 提取常量（如 `BASE_DISCOUNT_RATE`）  
- 阶段间数据传递清晰  

**效果**：使复杂流程变得**可追踪、可调试、可单元测试**。

## 三、贯穿始终的重构智慧

| 原则          | 说明                                             |
| ----------- | ---------------------------------------------- |
| **小步提交**    | 每次只做一件事，提交信息清晰（如 “extract calculate_discount”） |
| **测试驱动**    | 先写测试，再重构；每次提交后运行测试，确保行为不变                      |
| **命名即文档**   | 好名字胜过千行注释，命名要揭示“意图”而非“实现”                      |
| **避免过度抽象**  | 不要为每个小表达式都建类，**抽象要有价值**                        |
| **显式优于隐式**  | 控制流、数据流应清晰可见，避免“魔法”                            |
| **关注数据所有权** | 共享数据时，明确是“复制”还是“只读视图”                          |

## 四、重构避坑指南（Practical Gotchas）

> 以下是在实践中容易忽略但至关重要的细节：

- **拷贝 vs 引用**：封装数据时，调用方是否应获得副本？使用 `MappingProxyType` 可避免意外修改。
- **内联 ≠ 更好**：不要为了“减少行数”而内联，**保留有助于理解的中间变量**。
- **函数签名变更要渐进**：使用包装函数过渡，避免一次性破坏所有调用方。
- **测试要反映不变性**：重构可能改变返回结构（如字典键名），测试需同步更新。
- **职责分离**：不要将 UI、格式化、业务逻辑混在一起，**按阶段拆分**。

## 五、我的重构检查清单（Refactor Checklist）

**重构前**

- 已编写或更新测试，覆盖当前行为  
- 理解本次重构的目标（提升可读性？封装状态？）  
- 确认数据的共享与修改语义（复制 or 只读？）  

**重构中**

- 每次只做一个小改动，提交信息清晰  
- 使用有意义的命名，避免模糊术语  
- 优先使用小函数与清晰表达式  
- 运行测试，确保无行为变更  

**重构后**

- 代码是否更易读？能否让新人快速理解？  
- 控制流是否清晰？是否有隐藏的副作用？  
- 是否引入了不必要的抽象？  

## 六、结语：重构是一种思维习惯

通过本章的练习，我深刻体会到：

> **重构不是“等代码烂了再修”，而是“在每一次编码中持续优化”**。

Martin Fowler 所倡导的，是一种**以测试为盾、以小步为矛**的工程实践。它让我们在面对复杂系统时，依然能保持代码的**清晰、灵活与安全**。

这让我想起了最近的一个数据工程相关的项目，使用了大量的 Jupyter Notebooks 来运行 ETL 的各个环节。这种形式适于代码探索阶段，看看怎样的写法最适合。但如果在生产流水线上使用这样的 notebook，按我们会发现出现大量的坏味道：到处重复同一段代码，权责不明、巨型处理流程、 霰弹式修改等等。

所以在我刚上项目的 Onboarding 阶段就听得一头雾水，那么在拿到代码的第一时间，我就决定通过重构来熟悉代码，理解逻辑。重构不但能够帮我把代码组织得更好，而且还因为“测试先行”的原则，我可以通过构思测试用例来理解代码运行的 happy path，sad path 和副作用等等。当我把几百行的“数据接入”代码精简到一百来行，同时还生成了覆盖广泛的测试用例时，我就对于修改既有代码有了充足的信心。


[^1]: *本文由对《重构》第六章的代码实践与思考整理而成，力求还原重构的思维过程与工程价值。代码库：[Refactor-Practice](https://github.com/wang-zhenqi/Refactor-Practice)*