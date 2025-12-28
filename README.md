# 第二章：命令式编程 vs 声明式编程

## 🎯 核心目标

理解**命令式编程**与**声明式编程**的本质区别，掌握现代 Android 开发中声明式编程的思想要领。

---

## 📖 什么是命令式编程？

### 定义
**命令式编程（Imperative Programming）** 关注的是 **"怎么做（How）"**，即详细描述实现目标的每一个步骤。

### 特点
- ✅ 控制流程清晰，逐步执行
- ✅ 容易调试，步骤可追踪
- ❌ 代码冗长，需要关注细节
- ❌ 状态管理复杂，容易出错

### 示例：更新 UI 显示用户信息

```kotlin
// 命令式编程：手动操作 UI
fun updateUserUI(user: User) {
    // 步骤1：找到控件
    val nameTextView = findViewById<TextView>(R.id.tvName)
    val ageTextView = findViewById<TextView>(R.id.tvAge)
    val avatarImageView = findViewById<ImageView>(R.id.ivAvatar)
    
    // 步骤2：设置数据
    nameTextView.text = user.name
    ageTextView.text = "年龄: ${user.age}"
    
    // 步骤3：加载图片
    Glide.with(this)
        .load(user.avatarUrl)
        .into(avatarImageView)
    
    // 步骤4：根据条件显示/隐藏
    if (user.isVip) {
        vipBadge.visibility = View.VISIBLE
    } else {
        vipBadge.visibility = View.GONE
    }
}
```

**问题：**
- 需要手动管理每个控件的状态
- 代码与 UI 强耦合
- 如果有多个地方需要更新 UI，容易遗漏或重复

---

## 🌟 什么是声明式编程？

### 定义
**声明式编程（Declarative Programming）** 关注的是 **"要什么（What）"**，即描述期望的结果，而不是实现过程。

### 特点
- ✅ 代码简洁，专注于结果
- ✅ UI 与数据自动同步
- ✅ 可读性强，易于维护
- ❌ 需要适应新的思维方式

### 示例：使用 Jetpack Compose 实现相同功能

```kotlin
// 声明式编程：描述 UI 应该是什么样子
@Composable
fun UserProfile(user: User) {
    Column {
        AsyncImage(
            model = user.avatarUrl,
            contentDescription = "用户头像"
        )
        
        Text(text = user.name)
        Text(text = "年龄: ${user.age}")
        
        if (user.isVip) {
            VipBadge()
        }
    }
}
```

**优势：**
- 无需手动查找和更新控件
- UI 自动响应数据变化
- 代码更接近人的自然思维

---

## 🔄 核心区别对比

| 维度 | 命令式编程 | 声明式编程 |
|------|-----------|-----------|
| **关注点** | 怎么做（How） | 要什么（What） |
| **代码风格** | 详细的步骤描述 | 结果的直接描述 |
| **状态管理** | 手动管理每个状态变化 | 数据驱动，自动同步 |
| **UI 更新** | 手动调用 setter 方法 | 数据变化自动触发重组 |
| **可读性** | 代码冗长，需要理解流程 | 代码简洁，一目了然 |
| **典型代表** | Java + XML布局 + findViewById | Kotlin + Jetpack Compose |

---

## 💡 声明式编程的核心思想

### 1️⃣ **数据驱动 UI**
```kotlin
// 命令式：手动更新
var count = 0
button.setOnClickListener {
    count++
    textView.text = "点击次数: $count"  // 需要手动更新
}

// 声明式：数据变化自动反映
var count by remember { mutableStateOf(0) }
Button(onClick = { count++ }) {
    Text("点击次数: $count")  // 自动更新
}
```

### 2️⃣ **单向数据流**
```
用户操作 → 触发事件 → 更新状态 → UI 自动重组
   ↑                                    ↓
   └────────────── 显示新状态 ──────────┘
```

### 3️⃣ **组件化思维**
将 UI 拆分为独立、可复用的组件，每个组件只关注自己的数据和显示逻辑。

```kotlin
@Composable
fun UserCard(user: User) {
    Card {
        UserAvatar(user.avatarUrl)
        UserInfo(user.name, user.age)
        if (user.isVip) VipBadge()
    }
}
```

### 4️⃣ **不可变数据**
```kotlin
// 命令式：可变数据容易出错
val list = mutableListOf(1, 2, 3)
list.add(4)  // 可能在多线程中出问题

// 声明式：推荐不可变数据
val list = listOf(1, 2, 3)
val newList = list + 4  // 创建新列表，原列表不变
```

---

## 🎨 实战对比：购物车商品列表

### 命令式实现
```kotlin
class CartActivity : AppCompatActivity() {
    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: CartAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_cart)
        
        recyclerView = findViewById(R.id.rvCart)
        adapter = CartAdapter()
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter
        
        loadCartItems()
    }
    
    private fun loadCartItems() {
        viewModel.cartItems.observe(this) { items ->
            adapter.submitList(items)
        }
    }
    
    private fun updateTotalPrice(items: List<CartItem>) {
        val total = items.sumOf { it.price * it.quantity }
        findViewById<TextView>(R.id.tvTotal).text = "总价: ¥$total"
    }
}
```

### 声明式实现
```kotlin
@Composable
fun CartScreen(viewModel: CartViewModel) {
    val cartItems by viewModel.cartItems.collectAsState()
    
    Column {
        LazyColumn {
            items(cartItems) { item ->
                CartItemRow(item)
            }
        }
        
        Text(
            text = "总价: ¥${cartItems.sumOf { it.price * it.quantity }}",
            style = MaterialTheme.typography.h6
        )
    }
}
```

**对比：**
- 声明式代码减少约 50%
- 无需手动管理 RecyclerView 和 Adapter
- 总价自动计算和更新

---

## 🧠 思维方式转变

### 命令式思维：
> "我要先创建一个 TextView，然后设置它的文本，再设置颜色，最后添加到布局中"

### 声明式思维：
> "这里应该有一个红色的文本，内容是用户名"

```kotlin
// 声明式：直接描述结果
Text(
    text = user.name,
    color = Color.Red
)
```

---

## 📊 Android 中的声明式实践

| 场景 | 命令式 | 声明式 |
|------|--------|--------|
| **UI 构建** | XML + findViewById | Jetpack Compose |
| **列表展示** | RecyclerView + Adapter | LazyColumn/LazyRow |
| **状态管理** | 手动调用 setter | State + remember |
| **数据绑定** | DataBinding（半声明式） | Compose 自动绑定 |
| **动画** | Animation API | Compose Animation |

---

## ✅ 总结

1. **命令式编程**适合：
   - 需要精确控制流程的场景
   - 性能敏感的底层操作
   - 现有项目维护

2. **声明式编程**适合：
   - 现代 UI 开发
   - 复杂状态管理
   - 快速迭代的业务场景

3. **未来趋势**：
   - Android 官方推荐 Jetpack Compose（声明式）
   - 主流框架（React、Flutter、SwiftUI）都采用声明式
   - 声明式编程正在成为 UI 开发的主流

---

## 🎯 关键要点

💡 **记住这三句话：**
1. 命令式关注 **"怎么做"**，声明式关注 **"是什么"**
2. 声明式让 **数据驱动 UI**，而不是手动更新 UI
3. 用 **不可变数据 + 单向数据流** 让状态管理更简单

下一章我们将深入学习 Kotlin 的核心特性，为掌握声明式编程打下坚实基础！