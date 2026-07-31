# Firebase 配置（登录、云同步与在线人数）

本站使用三个 Firebase 功能：

- Authentication：Google 登录。
- Cloud Firestore：保存登录用户的学习进度。
- Realtime Database：显示页面最上方的绿色实时在线人数。

Firebase Web 配置不是密钥，可以放在前端。真正的数据保护来自 Firebase
Security Rules；不要把服务账号 JSON 或私钥提交到仓库。

## 1. 创建或确认 Web App

1. 打开 [Firebase Console](https://console.firebase.google.com/)，选择项目
   `i2dl-c79f8`（如果新建项目，则后面的项目 ID 和 URL 换成新值）。
2. 在“项目设置 → 常规 → 您的应用”中添加 Web App。
3. 复制 `firebaseConfig`，逐项填写到 `src/lib/config.ts`：
   `apiKey`、`authDomain`、`projectId`、`storageBucket`、
   `messagingSenderId`、`appId` 和 `databaseURL`。
4. 当前项目的 Realtime Database URL 应为：
   `https://i2dl-c79f8-default-rtdb.europe-west1.firebasedatabase.app`。

官方说明：[将 Firebase 添加到 JavaScript 项目](https://firebase.google.com/docs/web/setup)。

## 2. 开启 Google 登录（用于云同步）

1. Firebase Console → Authentication → Sign-in method。
2. 启用 Google，并选择项目支持邮箱。
3. Authentication → Settings → Authorized domains，确认包含：
   `c0nstantin77.github.io`；本地调试时也保留 `localhost`。

官方说明：[Web Google 登录](https://firebase.google.com/docs/auth/web/google-signin)。

## 3. 创建 Firestore（用于学习进度）

Firebase Console → Firestore Database → Create database。选好区域后，在 Rules
中至少保证每个用户只能读写自己的 `users/{uid}` 文档：

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, create, update, delete:
        if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

发布规则后再测试“Sign in to sync”。不要使用长期开放的测试模式。

## 4. 创建 Realtime Database（用于绿色在线人数）

1. Firebase Console → Realtime Database → Create database。
2. 选择数据库区域；若沿用现有配置，请使用当前 URL 对应的
   `europe-west1` 实例。
3. 初始模式可选择 Locked mode，然后立即部署仓库内的
   `database.rules.json`。

使用 Firebase CLI 部署：

```bash
npm install -g firebase-tools
firebase login
firebase use --add i2dl-c79f8
firebase deploy --only database
```

也可以在 Realtime Database → Rules 中粘贴 `database.rules.json` 的内容并
点击 Publish。规则只公开 `/presence` 的读取，并把写入限制为只有服务器时间戳
字段的连接记录；数据库其他路径默认拒绝访问。

官方说明：[创建并初始化 Realtime Database](https://firebase.google.com/docs/database/web/start)、
[Realtime Database Security Rules](https://firebase.google.com/docs/database/security)。

## 5. 清掉旧的错误在线记录

旧实现没有正确清理断线记录。Firebase Console → Realtime Database → Data，
选中根节点下的 `presence`，删除一次即可。新页面打开后会自动重新创建记录；
即使以后某次断线清理失败，前端也只统计最近 5 分钟内持续发送心跳的连接。

## 6. 部署并验证

```bash
npm run check
npm run build
```

把修改推送到 `main` 后，`.github/workflows/deploy.yml` 会构建并发布 GitHub
Pages。打开两个独立浏览器窗口：第一个应显示“only one”，第二个加入后两个窗口
都应显示“1 other person”。关闭第二个窗口后，人数通常会很快下降；异常断线最迟
约 5 分钟后不再计数。

这里统计的是活跃页面连接（一个人开两个标签页会算两个连接），不是登录账号数。
