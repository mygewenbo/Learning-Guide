# PyTorch 从零到精通 - 完整学习路线图

> **学习目标**: 系统掌握PyTorch深度学习框架，从基础到实战，具备独立开发深度学习项目的能力
> 
> **预计总时长**: 8-12周（每天2-3小时）
> 
> **前置要求**: Python基础、基本的线性代数和微积分知识

---

## 📚 学习路径总览

```
阶段1: 基础入门 (1-2周)
    ↓
阶段2: 核心概念 (2-3周)
    ↓
阶段3: 神经网络实战 (2-3周)
    ↓
阶段4: 高级特性 (2-3周)
    ↓
阶段5: 项目实战 (1-2周)
```

---

## 🎯 阶段1: PyTorch基础入门 (第1-2周)

### 1.1 环境搭建与配置 (第1天)

**学习目标**: 
- 成功安装PyTorch及相关依赖
- 验证GPU/CPU环境配置

**实践任务**:
```bash
# 安装PyTorch (根据你的系统选择)
pip install torch torchvision torchaudio

# 验证安装
python -c "import torch; print(torch.__version__)"
python -c "import torch; print(torch.cuda.is_available())"
```

**验证脚本**:
```python
import torch
import sys

print(f"PyTorch版本: {torch.__version__}")
print(f"Python版本: {sys.version}")
print(f"CUDA是否可用: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"CUDA版本: {torch.version.cuda}")
    print(f"GPU设备: {torch.cuda.get_device_name(0)}")
```

**学习资源**:
- PyTorch官方安装指南
- GPU驱动和CUDA工具包安装文档

---

### 1.2 张量(Tensor)基础操作 (第2-4天)

**学习目标**:
- 理解张量的概念和意义
- 掌握张量的创建、索引、切片
- 熟练进行张量运算

**核心知识点**:

#### 1.2.1 张量创建
```python
import torch

# 1. 从数据创建
data = [[1, 2], [3, 4]]
tensor = torch.tensor(data)

# 2. 从NumPy数组创建
import numpy as np
np_array = np.array([[1, 2], [3, 4]])
tensor_from_numpy = torch.from_numpy(np_array)

# 3. 使用工厂函数
zeros = torch.zeros(3, 4)          # 全0张量
ones = torch.ones(2, 3)            # 全1张量
rand = torch.rand(3, 3)            # 随机张量 [0,1)
randn = torch.randn(3, 3)          # 标准正态分布
arange = torch.arange(0, 10, 2)    # 等差序列
linspace = torch.linspace(0, 1, 5) # 线性间隔

# 4. 创建与其他张量相同形状的张量
x = torch.ones(2, 3)
y = torch.zeros_like(x)
z = torch.rand_like(x)
```

#### 1.2.2 张量属性
```python
tensor = torch.rand(3, 4, 5)

print(f"形状: {tensor.shape}")        # torch.Size([3, 4, 5])
print(f"维度: {tensor.ndim}")         # 3
print(f"数据类型: {tensor.dtype}")    # torch.float32
print(f"设备: {tensor.device}")       # cpu 或 cuda:0
print(f"元素总数: {tensor.numel()}")  # 60
```

#### 1.2.3 张量操作
```python
# 索引和切片
tensor = torch.arange(12).reshape(3, 4)
print(tensor[0])        # 第一行
print(tensor[:, 1])     # 第二列
print(tensor[1:, :2])   # 子矩阵

# 改变形状
x = torch.arange(12)
reshaped = x.reshape(3, 4)
viewed = x.view(2, 6)
unsqueezed = x.unsqueeze(0)  # 增加维度
squeezed = unsqueezed.squeeze()  # 压缩维度

# 拼接
x = torch.ones(2, 3)
y = torch.zeros(2, 3)
cat_0 = torch.cat([x, y], dim=0)  # 沿第0维拼接 -> (4, 3)
cat_1 = torch.cat([x, y], dim=1)  # 沿第1维拼接 -> (2, 6)
stacked = torch.stack([x, y], dim=0)  # 创建新维度 -> (2, 2, 3)
```

#### 1.2.4 数学运算
```python
x = torch.tensor([1.0, 2.0, 3.0])
y = torch.tensor([4.0, 5.0, 6.0])

# 基本运算（逐元素）
add = x + y              # 加法
sub = x - y              # 减法
mul = x * y              # 乘法
div = x / y              # 除法
power = x ** 2           # 幂运算

# 矩阵运算
A = torch.rand(3, 4)
B = torch.rand(4, 2)
matmul = torch.matmul(A, B)  # 矩阵乘法
# 或者使用 @ 运算符
matmul = A @ B

# 聚合操作
tensor = torch.rand(3, 4)
sum_all = tensor.sum()              # 所有元素求和
sum_dim0 = tensor.sum(dim=0)        # 沿第0维求和
mean_val = tensor.mean()            # 平均值
max_val = tensor.max()              # 最大值
min_val = tensor.min()              # 最小值
argmax = tensor.argmax()            # 最大值索引
```

**实践项目**:
```python
# 练习1: 实现图像的基本变换
def image_transform_practice():
    # 创建一个模拟的图像张量 (批次, 通道, 高, 宽)
    image = torch.rand(1, 3, 224, 224)
    
    # 任务1: 将图像归一化到[-1, 1]
    normalized = (image - 0.5) / 0.5
    
    # 任务2: 转换为灰度图（简单平均）
    grayscale = image.mean(dim=1, keepdim=True)
    
    # 任务3: 图像翻转
    flipped = torch.flip(image, dims=[3])  # 水平翻转
    
    return normalized, grayscale, flipped

# 练习2: 实现批量矩阵运算
def batch_matrix_operations():
    # 批量线性变换
    batch_size = 32
    input_features = 10
    output_features = 5
    
    X = torch.randn(batch_size, input_features)
    W = torch.randn(input_features, output_features)
    b = torch.randn(output_features)
    
    # Y = XW + b
    Y = torch.matmul(X, W) + b
    
    return Y
```

**作业**:
1. 创建一个 5x5 的随机矩阵，计算其转置、行列式、特征值
2. 实现一个函数，将任意形状的张量标准化 (均值0，标准差1)
3. 创建两个 3D 张量并进行批量矩阵乘法

---

### 1.3 GPU加速与设备管理 (第5-6天)

**学习目标**:
- 理解CPU与GPU的区别
- 掌握张量在不同设备间的转移
- 学会编写设备无关的代码

**核心知识点**:

```python
# 1. 检查设备可用性
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"使用设备: {device}")

# 2. 创建张量时指定设备
x_cpu = torch.rand(3, 3)                    # 默认在CPU
x_gpu = torch.rand(3, 3, device='cuda')     # 直接在GPU
x_gpu = torch.rand(3, 3).to(device)         # 使用.to()方法

# 3. 设备间转移
cpu_tensor = torch.rand(3, 3)
gpu_tensor = cpu_tensor.to('cuda')          # CPU -> GPU
back_to_cpu = gpu_tensor.cpu()              # GPU -> CPU
back_to_cpu = gpu_tensor.to('cpu')          # 等价方法

# 4. 多GPU管理
if torch.cuda.device_count() > 1:
    print(f"检测到 {torch.cuda.device_count()} 个GPU")
    tensor_gpu0 = torch.rand(3, 3, device='cuda:0')
    tensor_gpu1 = torch.rand(3, 3, device='cuda:1')

# 5. 设备无关代码模板
def train_model(model, data):
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = model.to(device)
    
    for batch in data:
        inputs = batch['input'].to(device)
        labels = batch['label'].to(device)
        # ... 训练逻辑
```

**性能对比实验**:
```python
import time

def benchmark_device(size=10000):
    # CPU运算
    start = time.time()
    x_cpu = torch.rand(size, size)
    y_cpu = torch.rand(size, size)
    z_cpu = torch.matmul(x_cpu, y_cpu)
    cpu_time = time.time() - start
    
    # GPU运算
    if torch.cuda.is_available():
        start = time.time()
        x_gpu = torch.rand(size, size, device='cuda')
        y_gpu = torch.rand(size, size, device='cuda')
        torch.cuda.synchronize()  # 等待GPU计算完成
        z_gpu = torch.matmul(x_gpu, y_gpu)
        torch.cuda.synchronize()
        gpu_time = time.time() - start
        
        print(f"CPU时间: {cpu_time:.4f}秒")
        print(f"GPU时间: {gpu_time:.4f}秒")
        print(f"加速比: {cpu_time/gpu_time:.2f}x")
```

---

### 1.4 张量的广播机制 (第7天)

**学习目标**:
- 理解广播的原理和规则
- 掌握不同形状张量的运算

**广播规则**:
1. 如果两个张量维度不同，在较小维度张量前面补1
2. 两个张量在某一维度上size相同，或其中一个为1，则兼容
3. 如果所有维度都兼容，则可以广播

```python
# 示例1: 标量与张量
x = torch.tensor([1, 2, 3])
y = 10
result = x + y  # [11, 12, 13]

# 示例2: 不同维度的张量
x = torch.rand(3, 1)      # (3, 1)
y = torch.rand(1, 4)      # (1, 4)
result = x + y            # (3, 4) 自动广播

# 示例3: 批量归一化场景
batch_data = torch.rand(32, 3, 224, 224)  # (batch, channel, H, W)
mean = torch.tensor([0.485, 0.456, 0.406]).reshape(1, 3, 1, 1)
std = torch.tensor([0.229, 0.224, 0.225]).reshape(1, 3, 1, 1)
normalized = (batch_data - mean) / std

# 示例4: 注意力机制中的掩码
scores = torch.rand(8, 10, 10)  # (batch, seq_len, seq_len)
mask = torch.tril(torch.ones(10, 10))  # (seq_len, seq_len)
masked_scores = scores.masked_fill(mask == 0, float('-inf'))
```

**作业**:
编写一个函数，实现batch normalization的广播计算

---

## 🔥 阶段2: 自动微分与核心概念 (第3-5周)

### 2.1 自动微分(Autograd) (第8-10天)

**学习目标**:
- 深入理解反向传播原理
- 掌握梯度计算和计算图
- 学会梯度管理技巧

**核心概念**:

#### 2.1.1 基础梯度计算
```python
import torch

# 启用梯度跟踪
x = torch.tensor([2.0, 3.0], requires_grad=True)
y = torch.tensor([1.0, 4.0], requires_grad=True)

# 前向传播：构建计算图
z = x ** 2 + 3 * y
loss = z.sum()

# 反向传播：计算梯度
loss.backward()

# 访问梯度
print(f"∂loss/∂x = {x.grad}")  # [4.0, 6.0]
print(f"∂loss/∂y = {y.grad}")  # [3.0, 3.0]

# 梯度清零（重要！）
x.grad.zero_()
y.grad.zero_()
```

#### 2.1.2 计算图与动态图
```python
# PyTorch使用动态计算图
def dynamic_computation(x, condition):
    if condition:
        y = x ** 2
    else:
        y = x ** 3
    return y

x = torch.tensor(2.0, requires_grad=True)

# 每次调用都会创建新的计算图
y1 = dynamic_computation(x, True)
y1.backward()
print(f"条件为True时的梯度: {x.grad}")

x.grad.zero_()
y2 = dynamic_computation(x, False)
y2.backward()
print(f"条件为False时的梯度: {x.grad}")
```

#### 2.1.3 梯度管理技巧
```python
# 1. 禁用梯度计算（推理时使用）
with torch.no_grad():
    y = x ** 2  # 不会构建计算图
    
# 2. 临时启用梯度
x = torch.tensor(2.0)  # requires_grad=False
with torch.enable_grad():
    x.requires_grad_(True)
    y = x ** 2

# 3. 阻断梯度传播
x = torch.tensor(2.0, requires_grad=True)
y = x ** 2
z = y.detach() * 3  # z不会接收梯度

# 4. 保留计算图（多次backward）
x = torch.tensor(2.0, requires_grad=True)
y = x ** 2
y.backward(retain_graph=True)  # 第一次
y.backward()  # 第二次

# 5. 计算高阶导数
x = torch.tensor(2.0, requires_grad=True)
y = x ** 3
grad_y = torch.autograd.grad(y, x, create_graph=True)[0]
grad2_y = torch.autograd.grad(grad_y, x)[0]  # 二阶导数
```

#### 2.1.4 自定义autograd函数
```python
class MyReLU(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input):
        ctx.save_for_backward(input)
        return input.clamp(min=0)
    
    @staticmethod
    def backward(ctx, grad_output):
        input, = ctx.saved_tensors
        grad_input = grad_output.clone()
        grad_input[input < 0] = 0
        return grad_input

# 使用自定义函数
x = torch.randn(5, requires_grad=True)
y = MyReLU.apply(x)
loss = y.sum()
loss.backward()
```

**实践项目**:
```python
# 项目1: 从零实现线性回归
class LinearRegression:
    def __init__(self, input_dim):
        self.w = torch.randn(input_dim, 1, requires_grad=True)
        self.b = torch.zeros(1, requires_grad=True)
    
    def forward(self, x):
        return torch.matmul(x, self.w) + self.b
    
    def train(self, X, y, lr=0.01, epochs=100):
        for epoch in range(epochs):
            # 前向传播
            y_pred = self.forward(X)
            loss = ((y_pred - y) ** 2).mean()
            
            # 反向传播
            loss.backward()
            
            # 更新参数
            with torch.no_grad():
                self.w -= lr * self.w.grad
                self.b -= lr * self.b.grad
                
                # 清零梯度
                self.w.grad.zero_()
                self.b.grad.zero_()
            
            if (epoch + 1) % 10 == 0:
                print(f'Epoch [{epoch+1}/{epochs}], Loss: {loss.item():.4f}')

# 测试
X = torch.randn(100, 3)
true_w = torch.tensor([[2.0], [-3.4], [1.5]])
true_b = torch.tensor([4.2])
y = torch.matmul(X, true_w) + true_b + torch.randn(100, 1) * 0.1

model = LinearRegression(3)
model.train(X, y)
```

---

### 2.2 数据加载与处理 (第11-13天)

**学习目标**:
- 掌握Dataset和DataLoader的使用
- 学会数据预处理和增强
- 理解批处理机制

#### 2.2.1 自定义Dataset
```python
from torch.utils.data import Dataset, DataLoader
import numpy as np

class CustomDataset(Dataset):
    def __init__(self, data, labels, transform=None):
        self.data = data
        self.labels = labels
        self.transform = transform
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        sample = self.data[idx]
        label = self.labels[idx]
        
        if self.transform:
            sample = self.transform(sample)
        
        return sample, label

# 使用示例
data = torch.randn(1000, 10)
labels = torch.randint(0, 2, (1000,))
dataset = CustomDataset(data, labels)

# 创建DataLoader
dataloader = DataLoader(
    dataset,
    batch_size=32,
    shuffle=True,
    num_workers=4,  # 多进程加载
    pin_memory=True  # 加速GPU传输
)

# 迭代数据
for batch_data, batch_labels in dataloader:
    print(f"Batch shape: {batch_data.shape}")
    break
```

#### 2.2.2 图像数据处理
```python
from torchvision import datasets, transforms

# 定义数据增强
transform_train = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), 
                        (0.2023, 0.1994, 0.2010))
])

transform_test = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), 
                        (0.2023, 0.1994, 0.2010))
])

# 加载CIFAR-10数据集
train_dataset = datasets.CIFAR10(
    root='./data',
    train=True,
    download=True,
    transform=transform_train
)

test_dataset = datasets.CIFAR10(
    root='./data',
    train=False,
    download=True,
    transform=transform_test
)

train_loader = DataLoader(train_dataset, batch_size=128, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=100, shuffle=False)
```

#### 2.2.3 自定义数据增强
```python
class AddGaussianNoise:
    def __init__(self, mean=0., std=0.1):
        self.mean = mean
        self.std = std
    
    def __call__(self, tensor):
        return tensor + torch.randn(tensor.size()) * self.std + self.mean

class Cutout:
    def __init__(self, n_holes, length):
        self.n_holes = n_holes
        self.length = length
    
    def __call__(self, img):
        h, w = img.size(1), img.size(2)
        mask = torch.ones((h, w), dtype=torch.float32)
        
        for _ in range(self.n_holes):
            y = torch.randint(h, (1,)).item()
            x = torch.randint(w, (1,)).item()
            
            y1 = max(0, y - self.length // 2)
            y2 = min(h, y + self.length // 2)
            x1 = max(0, x - self.length // 2)
            x2 = min(w, x + self.length // 2)
            
            mask[y1:y2, x1:x2] = 0.
        
        mask = mask.expand_as(img)
        return img * mask

# 组合使用
transform = transforms.Compose([
    transforms.ToTensor(),
    AddGaussianNoise(0., 0.1),
    Cutout(n_holes=1, length=16)
])
```

**作业**:
1. 创建一个文本数据集类，实现词汇表构建和序列填充
2. 实现一个自定义的数据增强策略（如Mixup）
3. 编写一个数据集分析工具，统计均值、标准差、类别分布

---

### 2.3 优化器与学习率调度 (第14天)

**学习目标**:
- 掌握常用优化器的使用
- 理解学习率调度策略
- 学会调参技巧

#### 2.3.1 常用优化器
```python
import torch.optim as optim

model = ... # 你的模型

# 1. SGD (随机梯度下降)
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9, weight_decay=5e-4)

# 2. Adam
optimizer = optim.Adam(model.parameters(), lr=0.001, betas=(0.9, 0.999))

# 3. AdamW (带权重衰减的Adam)
optimizer = optim.AdamW(model.parameters(), lr=0.001, weight_decay=0.01)

# 4. RMSprop
optimizer = optim.RMSprop(model.parameters(), lr=0.01, alpha=0.99)

# 5. Adagrad
optimizer = optim.Adagrad(model.parameters(), lr=0.01)

# 使用优化器
for epoch in range(num_epochs):
    for data, target in train_loader:
        optimizer.zero_grad()        # 清零梯度
        output = model(data)         # 前向传播
        loss = criterion(output, target)
        loss.backward()              # 反向传播
        optimizer.step()             # 更新参数
```

#### 2.3.2 学习率调度
```python
from torch.optim.lr_scheduler import *

# 1. StepLR: 每隔step_size个epoch衰减gamma倍
scheduler = StepLR(optimizer, step_size=30, gamma=0.1)

# 2. MultiStepLR: 在指定的epoch衰减
scheduler = MultiStepLR(optimizer, milestones=[30, 80], gamma=0.1)

# 3. ExponentialLR: 指数衰减
scheduler = ExponentialLR(optimizer, gamma=0.95)

# 4. CosineAnnealingLR: 余弦退火
scheduler = CosineAnnealingLR(optimizer, T_max=200)

# 5. ReduceLROnPlateau: 根据指标自适应调整
scheduler = ReduceLROnPlateau(optimizer, mode='min', factor=0.1, patience=10)

# 6. OneCycleLR: 单周期学习率策略
scheduler = OneCycleLR(optimizer, max_lr=0.1, steps_per_epoch=len(train_loader), epochs=10)

# 使用示例
for epoch in range(num_epochs):
    train(...)
    val_loss = validate(...)
    
    # 根据scheduler类型调用
    scheduler.step()  # StepLR, MultiStepLR等
    # scheduler.step(val_loss)  # ReduceLROnPlateau需要传入指标
```

#### 2.3.3 学习率预热(Warmup)
```python
class WarmupScheduler:
    def __init__(self, optimizer, warmup_epochs, base_lr, target_lr):
        self.optimizer = optimizer
        self.warmup_epochs = warmup_epochs
        self.base_lr = base_lr
        self.target_lr = target_lr
    
    def step(self, epoch):
        if epoch < self.warmup_epochs:
            lr = self.base_lr + (self.target_lr - self.base_lr) * epoch / self.warmup_epochs
            for param_group in self.optimizer.param_groups:
                param_group['lr'] = lr

# 使用
warmup = WarmupScheduler(optimizer, warmup_epochs=5, base_lr=1e-6, target_lr=1e-3)
for epoch in range(num_epochs):
    warmup.step(epoch)
    train(...)
```

---

## 🧠 阶段3: 神经网络构建与训练 (第4-6周)

### 3.1 nn.Module详解 (第15-17天)

**学习目标**:
- 掌握nn.Module的使用方法
- 学会构建自定义网络层
- 理解模型的保存与加载

#### 3.1.1 基础模型构建
```python
import torch.nn as nn
import torch.nn.functional as F

class SimpleNet(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super(SimpleNet, self).__init__()
        # 定义层
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, num_classes)
    
    def forward(self, x):
        # 定义前向传播
        out = self.fc1(x)
        out = self.relu(out)
        out = self.fc2(out)
        return out

# 实例化模型
model = SimpleNet(784, 128, 10)
print(model)

# 查看参数
for name, param in model.named_parameters():
    print(f"{name}: {param.shape}")
```

#### 3.1.2 常用层的使用
```python
# 1. 全连接层
fc = nn.Linear(in_features=100, out_features=50, bias=True)

# 2. 卷积层
conv2d = nn.Conv2d(in_channels=3, out_channels=64, kernel_size=3, stride=1, padding=1)
conv1d = nn.Conv1d(in_channels=256, out_channels=128, kernel_size=5)

# 3. 池化层
maxpool = nn.MaxPool2d(kernel_size=2, stride=2)
avgpool = nn.AdaptiveAvgPool2d((1, 1))

# 4. 归一化层
batchnorm1d = nn.BatchNorm1d(num_features=100)
batchnorm2d = nn.BatchNorm2d(num_features=64)
layernorm = nn.LayerNorm(normalized_shape=512)
instancenorm = nn.InstanceNorm2d(num_features=64)

# 5. Dropout
dropout = nn.Dropout(p=0.5)
dropout2d = nn.Dropout2d(p=0.2)

# 6. 激活函数
relu = nn.ReLU()
leaky_relu = nn.LeakyReLU(negative_slope=0.01)
sigmoid = nn.Sigmoid()
tanh = nn.Tanh()
gelu = nn.GELU()

# 7. 循环层
rnn = nn.RNN(input_size=100, hidden_size=256, num_layers=2, batch_first=True)
lstm = nn.LSTM(input_size=100, hidden_size=256, num_layers=2, batch_first=True)
gru = nn.GRU(input_size=100, hidden_size=256, num_layers=2, batch_first=True)

# 8. 注意力机制
multihead_attn = nn.MultiheadAttention(embed_dim=512, num_heads=8)

# 9. Transformer
transformer_layer = nn.TransformerEncoderLayer(d_model=512, nhead=8)
transformer = nn.TransformerEncoder(transformer_layer, num_layers=6)
```

#### 3.1.3 模型保存与加载
```python
# 1. 保存整个模型
torch.save(model, 'model.pth')
model = torch.load('model.pth')

# 2. 只保存参数（推荐）
torch.save(model.state_dict(), 'model_weights.pth')
model = SimpleNet(784, 128, 10)
model.load_state_dict(torch.load('model_weights.pth'))

# 3. 保存检查点（包含优化器状态）
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss,
}
torch.save(checkpoint, 'checkpoint.pth')

# 恢复训练
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

# 4. 跨设备保存和加载
# 保存在GPU上训练的模型
torch.save(model.state_dict(), 'model.pth')

# 加载到CPU
model.load_state_dict(torch.load('model.pth', map_location='cpu'))

# 加载到指定GPU
model.load_state_dict(torch.load('model.pth', map_location='cuda:1'))
```

---

### 3.2 卷积神经网络(CNN) (第18-21天)

**学习目标**:
- 理解卷积操作的原理
- 掌握经典CNN架构
- 实现图像分类任务

#### 3.2.1 卷积层详解
```python
# 卷积操作示例
import torch
import torch.nn as nn

# 输入: (batch, channels, height, width)
input_tensor = torch.randn(1, 3, 32, 32)

# 卷积层参数
conv = nn.Conv2d(
    in_channels=3,      # 输入通道数
    out_channels=64,    # 输出通道数（卷积核数量）
    kernel_size=3,      # 卷积核大小
    stride=1,           # 步长
    padding=1,          # 填充
    bias=True           # 是否使用偏置
)

output = conv(input_tensor)
print(f"输入形状: {input_tensor.shape}")
print(f"输出形状: {output.shape}")  # torch.Size([1, 64, 32, 32])

# 计算输出尺寸公式
# output_size = (input_size - kernel_size + 2*padding) / stride + 1
```

#### 3.2.2 实现LeNet-5
```python
class LeNet5(nn.Module):
    def __init__(self, num_classes=10):
        super(LeNet5, self).__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 6, kernel_size=5, padding=2),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=2, stride=2),
            nn.Conv2d(6, 16, kernel_size=5),
            nn.ReLU(),
            nn.MaxPool2d(kernel_size=2, stride=2)
        )
        self.classifier = nn.Sequential(
            nn.Linear(16 * 5 * 5, 120),
            nn.ReLU(),
            nn.Linear(120, 84),
            nn.ReLU(),
            nn.Linear(84, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)  # 展平
        x = self.classifier(x)
        return x

model = LeNet5()
x = torch.randn(32, 1, 28, 28)
output = model(x)
print(output.shape)  # torch.Size([32, 10])
```

#### 3.2.3 实现ResNet残差块
```python
class ResidualBlock(nn.Module):
    def __init__(self, in_channels, out_channels, stride=1):
        super(ResidualBlock, self).__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, 
                               stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3,
                               stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        self.shortcut = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, kernel_size=1, 
                         stride=stride, bias=False),
                nn.BatchNorm2d(out_channels)
            )
    
    def forward(self, x):
        residual = x
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += self.shortcut(residual)
        out = F.relu(out)
        return out

class ResNet(nn.Module):
    def __init__(self, block, num_blocks, num_classes=10):
        super(ResNet, self).__init__()
        self.in_channels = 64
        
        self.conv1 = nn.Conv2d(3, 64, kernel_size=3, stride=1, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.layer1 = self._make_layer(block, 64, num_blocks[0], stride=1)
        self.layer2 = self._make_layer(block, 128, num_blocks[1], stride=2)
        self.layer3 = self._make_layer(block, 256, num_blocks[2], stride=2)
        self.layer4 = self._make_layer(block, 512, num_blocks[3], stride=2)
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512, num_classes)
    
    def _make_layer(self, block, out_channels, num_blocks, stride):
        strides = [stride] + [1] * (num_blocks - 1)
        layers = []
        for stride in strides:
            layers.append(block(self.in_channels, out_channels, stride))
            self.in_channels = out_channels
        return nn.Sequential(*layers)
    
    def forward(self, x):
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.layer1(out)
        out = self.layer2(out)
        out = self.layer3(out)
        out = self.layer4(out)
        out = self.avgpool(out)
        out = out.view(out.size(0), -1)
        out = self.fc(out)
        return out

# ResNet-18
model = ResNet(ResidualBlock, [2, 2, 2, 2])
```

#### 3.2.4 完整的训练脚本
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms

def train_epoch(model, train_loader, criterion, optimizer, device):
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0
    
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)
        
        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()
        
        running_loss += loss.item()
        _, predicted = output.max(1)
        total += target.size(0)
        correct += predicted.eq(target).sum().item()
    
    epoch_loss = running_loss / len(train_loader)
    epoch_acc = 100. * correct / total
    return epoch_loss, epoch_acc

def validate(model, test_loader, criterion, device):
    model.eval()
    test_loss = 0.0
    correct = 0
    total = 0
    
    with torch.no_grad():
        for data, target in test_loader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            loss = criterion(output, target)
            
            test_loss += loss.item()
            _, predicted = output.max(1)
            total += target.size(0)
            correct += predicted.eq(target).sum().item()
    
    test_loss /= len(test_loader)
    test_acc = 100. * correct / total
    return test_loss, test_acc

# 主训练循环
def train_model(model, train_loader, test_loader, num_epochs=10):
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = model.to(device)
    
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)
    scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)
    
    best_acc = 0.0
    
    for epoch in range(num_epochs):
        train_loss, train_acc = train_epoch(model, train_loader, criterion, optimizer, device)
        test_loss, test_acc = validate(model, test_loader, criterion, device)
        
        print(f'Epoch {epoch+1}/{num_epochs}')
        print(f'Train Loss: {train_loss:.4f}, Train Acc: {train_acc:.2f}%')
        print(f'Test Loss: {test_loss:.4f}, Test Acc: {test_acc:.2f}%')
        
        # 保存最佳模型
        if test_acc > best_acc:
            best_acc = test_acc
            torch.save(model.state_dict(), 'best_model.pth')
        
        scheduler.step()
    
    print(f'Best Test Accuracy: {best_acc:.2f}%')
```

**实践项目**:
1. 在CIFAR-10数据集上训练ResNet-18
2. 实现数据增强并对比效果
3. 使用预训练模型进行迁移学习

---

### 3.3 循环神经网络(RNN/LSTM/GRU) (第22-24天)

**学习目标**:
- 理解序列建模的原理
- 掌握RNN、LSTM、GRU的使用
- 实现文本分类和序列预测

#### 3.3.1 基础RNN实现
```python
class SimpleRNN(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, num_classes):
        super(SimpleRNN, self).__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.rnn = nn.RNN(input_size, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, num_classes)
    
    def forward(self, x):
        # x: (batch, seq_len, input_size)
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        out, _ = self.rnn(x, h0)
        # out: (batch, seq_len, hidden_size)
        out = self.fc(out[:, -1, :])  # 取最后一个时间步
        return out
```

#### 3.3.2 LSTM实现
```python
class LSTMModel(nn.Module):
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_layers, num_classes, dropout=0.5):
        super(LSTMModel, self).__init__()
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.lstm = nn.LSTM(embedding_dim, hidden_size, num_layers, 
                           batch_first=True, dropout=dropout)
        self.fc = nn.Linear(hidden_size, num_classes)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x):
        # x: (batch, seq_len)
        embedded = self.dropout(self.embedding(x))
        # embedded: (batch, seq_len, embedding_dim)
        
        lstm_out, (hidden, cell) = self.lstm(embedded)
        # lstm_out: (batch, seq_len, hidden_size)
        # hidden: (num_layers, batch, hidden_size)
        
        # 使用最后一层的隐藏状态
        output = self.fc(hidden[-1])
        return output

# 双向LSTM
class BiLSTM(nn.Module):
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_layers, num_classes):
        super(BiLSTM, self).__init__()
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.bilstm = nn.LSTM(embedding_dim, hidden_size, num_layers,
                             batch_first=True, bidirectional=True)
        self.fc = nn.Linear(hidden_size * 2, num_classes)  # *2 因为是双向
    
    def forward(self, x):
        embedded = self.embedding(x)
        lstm_out, (hidden, cell) = self.bilstm(embedded)
        # 拼接前向和后向的最后隐藏状态
        hidden = torch.cat((hidden[-2,:,:], hidden[-1,:,:]), dim=1)
        output = self.fc(hidden)
        return output
```

#### 3.3.3 序列到序列(Seq2Seq)模型
```python
class Seq2Seq(nn.Module):
    def __init__(self, input_dim, embedding_dim, hidden_dim, output_dim):
        super(Seq2Seq, self).__init__()
        
        # 编码器
        self.encoder_embedding = nn.Embedding(input_dim, embedding_dim)
        self.encoder_lstm = nn.LSTM(embedding_dim, hidden_dim, batch_first=True)
        
        # 解码器
        self.decoder_embedding = nn.Embedding(output_dim, embedding_dim)
        self.decoder_lstm = nn.LSTM(embedding_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)
    
    def forward(self, src, trg):
        # 编码
        embedded_src = self.encoder_embedding(src)
        _, (hidden, cell) = self.encoder_lstm(embedded_src)
        
        # 解码
        embedded_trg = self.decoder_embedding(trg)
        decoder_output, _ = self.decoder_lstm(embedded_trg, (hidden, cell))
        output = self.fc(decoder_output)
        
        return output
```

**实践项目**:
1. 实现情感分类模型（IMDB数据集）
2. 实现序列标注任务（命名实体识别）
3. 实现简单的机器翻译模型

---

## 🚀 阶段4: 高级技术与特性 (第7-9周)

### 4.1 迁移学习与预训练模型 (第25-27天)

**学习目标**:
- 掌握迁移学习的原理和方法
- 学会使用torchvision预训练模型
- 实现Fine-tuning技巧

#### 4.1.1 使用预训练模型
```python
import torchvision.models as models

# 1. 加载预训练模型
resnet50 = models.resnet50(pretrained=True)
vgg16 = models.vgg16(pretrained=True)
efficientnet = models.efficientnet_b0(pretrained=True)

# 2. 特征提取模式（冻结参数）
model = models.resnet50(pretrained=True)
for param in model.parameters():
    param.requires_grad = False

# 修改最后一层
num_features = model.fc.in_features
model.fc = nn.Linear(num_features, 10)  # 10个类别

# 3. Fine-tuning模式（部分解冻）
# 冻结前面的层
for name, param in model.named_parameters():
    if 'layer4' not in name and 'fc' not in name:
        param.requires_grad = False

# 4. 不同层使用不同学习率
params_to_update = []
params_to_update_fc = []

for name, param in model.named_parameters():
    if param.requires_grad:
        if 'fc' in name:
            params_to_update_fc.append(param)
        else:
            params_to_update.append(param)

optimizer = optim.SGD([
    {'params': params_to_update, 'lr': 0.001},
    {'params': params_to_update_fc, 'lr': 0.01}
], momentum=0.9)
```

#### 4.1.2 自定义预训练模型的使用
```python
class TransferLearningModel(nn.Module):
    def __init__(self, num_classes, freeze_backbone=True):
        super(TransferLearningModel, self).__init__()
        # 加载预训练的ResNet
        backbone = models.resnet50(pretrained=True)
        
        if freeze_backbone:
            for param in backbone.parameters():
                param.requires_grad = False
        
        # 提取特征提取部分
        self.features = nn.Sequential(*list(backbone.children())[:-1])
        
        # 自定义分类头
        self.classifier = nn.Sequential(
            nn.Dropout(0.5),
            nn.Linear(2048, 512),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(512, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        x = self.classifier(x)
        return x

# 渐进式解冻训练
def progressive_unfreezing(model, optimizer, train_loader, epochs_per_stage=5):
    stages = [
        ['layer4', 'classifier'],
        ['layer3', 'layer4', 'classifier'],
        ['layer2', 'layer3', 'layer4', 'classifier'],
    ]
    
    for stage, layers_to_train in enumerate(stages):
        print(f"Stage {stage + 1}: Training {layers_to_train}")
        
        # 冻结所有参数
        for param in model.parameters():
            param.requires_grad = False
        
        # 解冻指定层
        for name, param in model.named_parameters():
            if any(layer in name for layer in layers_to_train):
                param.requires_grad = True
        
        # 训练该阶段
        for epoch in range(epochs_per_stage):
            train_epoch(model, train_loader, criterion, optimizer, device)
```

---

### 4.2 模型部署与优化 (第28-30天)

**学习目标**:
- 掌握模型量化和剪枝技术
- 学习TorchScript和ONNX导出
- 了解模型加速技巧

#### 4.2.1 模型量化
```python
import torch.quantization as quantization

# 1. 动态量化（最简单）
model_fp32 = MyModel()
model_int8 = torch.quantization.quantize_dynamic(
    model_fp32, 
    {nn.Linear}, 
    dtype=torch.qint8
)

# 2. 静态量化
model_fp32.eval()
model_fp32.qconfig = torch.quantization.get_default_qconfig('fbgemm')
model_fp32_prepared = torch.quantization.prepare(model_fp32)

# 校准
with torch.no_grad():
    for data, _ in calibration_loader:
        model_fp32_prepared(data)

model_int8 = torch.quantization.convert(model_fp32_prepared)

# 3. 量化感知训练(QAT)
model_fp32.qconfig = torch.quantization.get_default_qat_qconfig('fbgemm')
model_fp32_qat = torch.quantization.prepare_qat(model_fp32)

# 训练
for epoch in range(num_epochs):
    train(model_fp32_qat, train_loader, optimizer)

model_int8 = torch.quantization.convert(model_fp32_qat)
```

#### 4.2.2 TorchScript导出
```python
# 1. 使用torch.jit.trace
model = MyModel()
model.eval()

example_input = torch.rand(1, 3, 224, 224)
traced_script_module = torch.jit.trace(model, example_input)

# 保存
traced_script_module.save("model_traced.pt")

# 加载
loaded_model = torch.jit.load("model_traced.pt")

# 2. 使用torch.jit.script
@torch.jit.script
def my_function(x):
    if x.sum() > 0:
        return x * 2
    else:
        return x / 2

class MyScriptModule(torch.nn.Module):
    def __init__(self):
        super(MyScriptModule, self).__init__()
        self.conv = nn.Conv2d(3, 64, 3)
    
    def forward(self, x):
        return F.relu(self.conv(x))

scripted_module = torch.jit.script(MyScriptModule())
scripted_module.save("model_scripted.pt")
```

#### 4.2.3 ONNX导出
```python
import torch.onnx

model = MyModel()
model.eval()

dummy_input = torch.randn(1, 3, 224, 224)

# 导出ONNX
torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    export_params=True,
    opset_version=11,
    do_constant_folding=True,
    input_names=['input'],
    output_names=['output'],
    dynamic_axes={
        'input': {0: 'batch_size'},
        'output': {0: 'batch_size'}
    }
)

# 验证ONNX模型
import onnx
onnx_model = onnx.load("model.onnx")
onnx.checker.check_model(onnx_model)

# 使用ONNX Runtime推理
import onnxruntime as ort
ort_session = ort.InferenceSession("model.onnx")
outputs = ort_session.run(
    None,
    {"input": dummy_input.numpy()}
)
```

---

### 4.3 高级训练技巧 (第31-33天)

**学习目标**:
- 掌握混合精度训练
- 学习梯度裁剪和梯度累积
- 了解对抗训练和正则化

#### 4.3.1 混合精度训练
```python
from torch.cuda.amp import autocast, GradScaler

# 创建GradScaler
scaler = GradScaler()

for epoch in range(num_epochs):
    for data, target in train_loader:
        data, target = data.cuda(), target.cuda()
        
        optimizer.zero_grad()
        
        # 自动混合精度
        with autocast():
            output = model(data)
            loss = criterion(output, target)
        
        # 缩放损失并反向传播
        scaler.scale(loss).backward()
        
        # 梯度裁剪（可选）
        scaler.unscale_(optimizer)
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        
        # 更新权重
        scaler.step(optimizer)
        scaler.update()
```

#### 4.3.2 梯度累积
```python
accumulation_steps = 4

optimizer.zero_grad()
for i, (data, target) in enumerate(train_loader):
    data, target = data.to(device), target.to(device)
    
    output = model(data)
    loss = criterion(output, target)
    
    # 归一化损失
    loss = loss / accumulation_steps
    loss.backward()
    
    # 每accumulation_steps步更新一次
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad()
```

#### 4.3.3 早停和模型检查点
```python
class EarlyStopping:
    def __init__(self, patience=7, min_delta=0, path='checkpoint.pth'):
        self.patience = patience
        self.min_delta = min_delta
        self.path = path
        self.counter = 0
        self.best_loss = None
        self.early_stop = False
    
    def __call__(self, val_loss, model):
        if self.best_loss is None:
            self.best_loss = val_loss
            self.save_checkpoint(model)
        elif val_loss > self.best_loss - self.min_delta:
            self.counter += 1
            if self.counter >= self.patience:
                self.early_stop = True
        else:
            self.best_loss = val_loss
            self.save_checkpoint(model)
            self.counter = 0
    
    def save_checkpoint(self, model):
        torch.save(model.state_dict(), self.path)

# 使用
early_stopping = EarlyStopping(patience=10)

for epoch in range(num_epochs):
    train_loss = train_epoch(model, train_loader, optimizer, criterion)
    val_loss = validate(model, val_loader, criterion)
    
    early_stopping(val_loss, model)
    
    if early_stopping.early_stop:
        print("Early stopping triggered")
        break
```

---

### 4.4 Transformer与注意力机制 (第34-36天)

**学习目标**:
- 理解注意力机制原理
- 实现Transformer模型
- 掌握位置编码

#### 4.4.1 自注意力机制
```python
class SelfAttention(nn.Module):
    def __init__(self, embed_size, heads):
        super(SelfAttention, self).__init__()
        self.embed_size = embed_size
        self.heads = heads
        self.head_dim = embed_size // heads
        
        assert (self.head_dim * heads == embed_size), "Embed size needs to be divisible by heads"
        
        self.values = nn.Linear(self.head_dim, self.head_dim, bias=False)
        self.keys = nn.Linear(self.head_dim, self.head_dim, bias=False)
        self.queries = nn.Linear(self.head_dim, self.head_dim, bias=False)
        self.fc_out = nn.Linear(heads * self.head_dim, embed_size)
    
    def forward(self, values, keys, query, mask=None):
        N = query.shape[0]
        value_len, key_len, query_len = values.shape[1], keys.shape[1], query.shape[1]
        
        # 分割成多头
        values = values.reshape(N, value_len, self.heads, self.head_dim)
        keys = keys.reshape(N, key_len, self.heads, self.head_dim)
        queries = query.reshape(N, query_len, self.heads, self.head_dim)
        
        values = self.values(values)
        keys = self.keys(keys)
        queries = self.queries(queries)
        
        # 计算注意力分数
        energy = torch.einsum("nqhd,nkhd->nhqk", [queries, keys])
        
        if mask is not None:
            energy = energy.masked_fill(mask == 0, float("-1e20"))
        
        attention = torch.softmax(energy / (self.embed_size ** (1/2)), dim=3)
        
        out = torch.einsum("nhql,nlhd->nqhd", [attention, values]).reshape(
            N, query_len, self.heads * self.head_dim
        )
        
        out = self.fc_out(out)
        return out
```

#### 4.4.2 位置编码
```python
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super(PositionalEncoding, self).__init__()
        
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        x = x + self.pe[:, :x.size(1), :]
        return x
```

#### 4.4.3 完整Transformer编码器
```python
import math

class TransformerBlock(nn.Module):
    def __init__(self, embed_size, heads, dropout, forward_expansion):
        super(TransformerBlock, self).__init__()
        self.attention = SelfAttention(embed_size, heads)
        self.norm1 = nn.LayerNorm(embed_size)
        self.norm2 = nn.LayerNorm(embed_size)
        
        self.feed_forward = nn.Sequential(
            nn.Linear(embed_size, forward_expansion * embed_size),
            nn.ReLU(),
            nn.Linear(forward_expansion * embed_size, embed_size)
        )
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, value, key, query, mask):
        attention = self.attention(value, key, query, mask)
        x = self.dropout(self.norm1(attention + query))
        forward = self.feed_forward(x)
        out = self.dropout(self.norm2(forward + x))
        return out

class TransformerEncoder(nn.Module):
    def __init__(self, vocab_size, embed_size, num_layers, heads, 
                 forward_expansion, dropout, max_length):
        super(TransformerEncoder, self).__init__()
        self.embed_size = embed_size
        self.word_embedding = nn.Embedding(vocab_size, embed_size)
        self.position_embedding = nn.Embedding(max_length, embed_size)
        
        self.layers = nn.ModuleList([
            TransformerBlock(embed_size, heads, dropout, forward_expansion)
            for _ in range(num_layers)
        ])
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, mask):
        N, seq_length = x.shape
        positions = torch.arange(0, seq_length).expand(N, seq_length).to(x.device)
        
        out = self.dropout(
            self.word_embedding(x) + self.position_embedding(positions)
        )
        
        for layer in self.layers:
            out = layer(out, out, out, mask)
        
        return out
```

---

## 💼 阶段5: 综合项目实战 (第10-12周)

### 5.1 计算机视觉项目 (第37-39天)

**项目1: 图像分类系统**
- 数据集：自定义数据集或Kaggle竞赛数据
- 任务：实现从数据加载到模型部署的完整流程
- 技术点：数据增强、迁移学习、模型集成

**项目2: 目标检测**
- 使用torchvision实现Faster R-CNN或YOLO
- 在COCO或VOC数据集上训练
- 实现实时检测系统

**项目3: 图像分割**
- 实现U-Net或DeepLab
- 语义分割或实例分割任务
- 医学图像或卫星图像分割

---

### 5.2 自然语言处理项目 (第40-42天)

**项目1: 文本分类**
- 情感分析或主题分类
- 使用LSTM/GRU或Transformer
- 部署为REST API

**项目2: 命名实体识别**
- 使用BiLSTM-CRF模型
- 中文或英文NER任务
- 实现实时标注系统

**项目3: 文本生成**
- 实现简单的GPT风格模型
- 诗歌生成或对话生成
- 使用预训练模型Fine-tuning

---

### 5.3 推荐系统项目 (第43-44天)

**项目: 协同过滤推荐系统**
- 实现矩阵分解或神经协同过滤
- 在MovieLens数据集上训练
- 实现TopN推荐

---

### 5.4 生成对抗网络(GAN)项目 (第45-46天)

**项目: 图像生成**
```python
# DCGAN示例
class Generator(nn.Module):
    def __init__(self, nz, ngf, nc):
        super(Generator, self).__init__()
        self.main = nn.Sequential(
            nn.ConvTranspose2d(nz, ngf * 8, 4, 1, 0, bias=False),
            nn.BatchNorm2d(ngf * 8),
            nn.ReLU(True),
            nn.ConvTranspose2d(ngf * 8, ngf * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf * 4),
            nn.ReLU(True),
            nn.ConvTranspose2d(ngf * 4, ngf * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf * 2),
            nn.ReLU(True),
            nn.ConvTranspose2d(ngf * 2, ngf, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ngf),
            nn.ReLU(True),
            nn.ConvTranspose2d(ngf, nc, 4, 2, 1, bias=False),
            nn.Tanh()
        )
    
    def forward(self, input):
        return self.main(input)

class Discriminator(nn.Module):
    def __init__(self, nc, ndf):
        super(Discriminator, self).__init__()
        self.main = nn.Sequential(
            nn.Conv2d(nc, ndf, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(ndf, ndf * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ndf * 2),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(ndf * 2, ndf * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(ndf * 4),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(ndf * 4, 1, 4, 1, 0, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, input):
        return self.main(input)
```

---

## 📖 学习资源推荐

### 官方文档
- **PyTorch官方文档**: https://pytorch.org/docs/
- **PyTorch Tutorials**: https://pytorch.org/tutorials/
- **torchvision文档**: https://pytorch.org/vision/

### 在线课程
- **Fast.ai**: Practical Deep Learning for Coders
- **Coursera**: Deep Learning Specialization (Andrew Ng)
- **Udacity**: Deep Learning Nanodegree

### 书籍推荐
- 《Deep Learning with PyTorch》
- 《Programming PyTorch for Deep Learning》
- 《PyTorch深度学习实战》

### 社区资源
- **PyTorch论坛**: https://discuss.pytorch.org/
- **GitHub**: pytorch/pytorch, pytorch/examples
- **Papers with Code**: 最新论文和实现

---

## ✅ 学习检查清单

### 基础部分
- [ ] 熟练创建和操作张量
- [ ] 理解GPU加速原理
- [ ] 掌握自动微分机制
- [ ] 会使用DataLoader加载数据

### 模型构建
- [ ] 能够使用nn.Module构建模型
- [ ] 理解常用层的作用
- [ ] 掌握模型保存和加载
- [ ] 会使用预训练模型

### 训练技巧
- [ ] 掌握常用优化器
- [ ] 会使用学习率调度
- [ ] 理解正则化方法
- [ ] 能够调试训练过程

### 高级特性
- [ ] 掌握混合精度训练
- [ ] 了解模型量化
- [ ] 会导出ONNX模型
- [ ] 理解分布式训练

### 项目实战
- [ ] 完成至少3个完整项目
- [ ] 能够部署模型到生产
- [ ] 掌握模型优化技巧
- [ ] 有阅读论文实现的能力

---

## 🎯 每日学习建议

### 时间安排
- **理论学习**: 30-45分钟
- **代码实践**: 60-90分钟
- **项目练习**: 30-45分钟
- **总结复习**: 15-30分钟

### 学习方法
1. **主动实践**: 每个知识点都要亲手编写代码
2. **记录笔记**: 整理重点和踩过的坑
3. **查阅文档**: 养成查看官方文档的习惯
4. **社区交流**: 参与论坛讨论，学习他人经验
5. **项目驱动**: 以项目为导向，学以致用

### 常见问题解决
- **CUDA out of memory**: 减小batch size，使用混合精度训练
- **梯度爆炸/消失**: 使用梯度裁剪，调整学习率
- **过拟合**: 增加数据增强，使用Dropout，正则化
- **训练太慢**: 检查数据加载，使用多进程，GPU加速

---

## 🏆 进阶方向

完成基础学习后，可以根据兴趣选择以下方向深入：

1. **计算机视觉**: 目标检测、图像分割、3D视觉
2. **自然语言处理**: 大语言模型、机器翻译、问答系统
3. **强化学习**: DQN、PPO、AlphaGo算法
4. **图神经网络**: GCN、GAT、图分类任务
5. **模型压缩**: 量化、剪枝、知识蒸馏
6. **分布式训练**: DDP、FSDP、大规模训练

---

## 📝 总结

这份学习计划涵盖了从PyTorch基础到高级应用的完整路径。建议：

1. **循序渐进**: 不要跳跃式学习，打牢基础很重要
2. **持之以恒**: 每天保持2-3小时的高质量学习时间
3. **多动手练**: 理论结合实践，代码是最好的老师
4. **及时总结**: 定期回顾所学内容，建立知识体系
5. **参与社区**: 与他人交流，共同进步

祝您学习顺利，早日成为PyTorch高手！🚀
