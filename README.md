# Zectrix 墨水屏便利贴 日历同步

自动将邮箱 CalDAV日历同步到 Zectrix 极趣墨水屏电子便利贴。

https://space.bilibili.com/13131424?spm_id_from=333.788.upinfo.detail.click
https://cloud.zectrix.com/

## 功能

- 🧹 **自动清理过期日程**: 已超过1小时的 `[日历]` 开头日程自动删除
- 📅 **同步未来日程**: 只同步未来24小时内的日程，保持屏幕整洁
- 🚫 **过滤已取消**: 自动跳过标题包含"已取消"、"cancelled"的会议
- 🔁 **自动重试**: 网络请求失败自动重试（最多3次，指数退避）
- 🎯 **去重**: 已存在的日程不会重复创建

## 工作流程

1. **第一步** - 调用 Zectrix 极趣云API 查询所有未完成待办
2. **第二步** - 删除满足两个条件的待办:
   - Title 以 `[日历]` 开头
   - 日程时间 `dueDate + dueTime` 距离现在已经超过 1 小时
3. **第三步** - 通过 CalDAV 从邮箱获取未来 24 小时内的日历事件
4. **第四步** - 对于日历中的每个日程:
   - 如果按 `标题` + `日期` 在现有列表中找不到 → 调用创建接口新建，title自动加上 `[日历]` 前缀
   - 如果已存在 → 跳过

## 依赖安装

```bash
pip install caldav icalendar requests
```

## 配置

所有配置通过**环境变量**注入，支持两种方式：

### 方式一：临时设置环境变量运行

```bash
# Zectrix API 配置
export API_BASE="https://cloud.zectrix.com/open/v1"
export API_KEY="your-api-key"          # 你的 API Key
export DEVICE_ID="your-device-id"      # 你的设备 ID
export EXPIRE_HOURS="1"                # 过期多久后删除，单位小时

# CalDAV 配置
export CALDAV_URL="https://caldav.mxhichina.com/dav/your-email@example.com/"
export CALDAV_USER="your-email@example.com"  # 邮箱用户名
export CALDAV_PASS="your-password"           # 邮箱密码/授权码

python sync_calendar.py
```

### 方式二：通过 `.env` 文件配置（推荐）

创建 `.env` 文件：

```env
# Zectrix API 配置
API_BASE=https://cloud.zectrix.com/open/v1
API_KEY=your-api-key
DEVICE_ID=your-device-id
EXPIRE_HOURS=1

# CalDAV 配置
CALDAV_URL=https://caldav.mxhichina.com/dav/your-email@example.com/
CALDAV_USER=your-email@example.com
CALDAV_PASS=your-password
```

然后使用 python-dotenv 加载：

```bash
pip install python-dotenv
```

在运行前加载：

```bash
python -c "import dotenv; dotenv.load_dotenv(); import sync_calendar"
```

或者如果你使用 bash：

```bash
export $(grep -v '^#' .env | xargs) && python sync_calendar.py
```

### 环境变量说明

| 环境变量 | 必填 | 默认值 | 说明 |
|---------|------|--------|------|
| `API_BASE` | 否 | `https://cloud.zectrix.com/open/v1` | Zectrix API 地址 |
| `API_KEY` | 是 | - | 你的 API Key，从 Zectrix 云端获取 |
| `DEVICE_ID` | 是 | - | 你的设备 ID |
| `EXPIRE_HOURS` | 否 | `1` | 过期多久（小时）后删除过期日程 |
| `CALDAV_URL` | 否 | 阿里云邮箱默认地址 | CalDAV 服务器地址 |
| `CALDAV_USER` | 是 | - | CalDAV 用户名（通常是邮箱地址） |
| `CALDAV_PASS` | 是 | - | CalDAV 密码/授权码 |

## 运行

```bash
python sync_calendar.py
```

## 定时运行（推荐）

可以用 crontab 定时同步，比如每小时同步一次:

```bash
crontab -e
```

添加一行:

```
0 * * * * /usr/bin/python3 /path/to/sync_calendar.py >> /var/log/zectrix_sync.log 2>&1
```

## 说明

- 时区自动处理，CalDAV 中的 UTC 时间会正确转换为本地时区
- 全天事件默认按当天 09:00 处理
- 只有未取消且在未来24小时内的日程才会被同步

## License

MIT
