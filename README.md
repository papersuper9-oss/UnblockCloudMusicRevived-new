# UnblockCloudMusicRevived-new
网易云面V1P插件，网易云版本2.10.13，BetterNCM 1.3.4，适用：Windows 10/11 64 位 + 网易云音乐 + BetterNCM 版本：UNM enhanced 分支源码构建（2026-10-29）
--------------------------------------------------------
 一、安装步骤
--------------------------------------------------------

1) 确认目标电脑已装好 BetterNCM（能打开插件市场即可）。

2) 把本压缩包**完整解压**到任意目录（不要在压缩包里直接运行）。

3) 在解压目录空白处 Shift+右键 → “在此处打开 PowerShell 窗口”，执行：

     powershell -ExecutionPolicy Bypass -File .\install.ps1

   如果 BetterNCM 数据目录不是 C:\betterncm，脚本会自动尝试
   %APPDATA%\BetterNCM 等位置；也可以手动指定：

     powershell -ExecutionPolicy Bypass -File .\install.ps1 -DataPath "D:\betterncm"

4) 脚本会：关闭网易云 → 复制程序 → 安装插件包 → 写配置 → 联网自检。
   自检显示 “取到真实音频 xx MB（晴天/320k）” 即为成功。

5) 启动网易云音乐，播放任意 VIP 歌曲验证。

--------------------------------------------------------
 二、这个包为什么这么装
--------------------------------------------------------

BetterNCM 的 RevivedUnblockInstaller 插件只会启动 UnblockNeteaseMusic-*.exe，
而官方最后一个 release（v0.28.0，2025-09）的音源已经全部失效——酷我会给所有歌
返回同一个 185KB 的推广音频（“酷我音乐为你开启免费听歌权限…”），酷狗/B站/咪咕
也都取不到。仓库 enhanced 分支一直在修（2026-10 仍在提交），但不发新 release。

所以本包的做法是：
  · 直接跑 enhanced 分支源码（server-src/，内置便携 node，无需安装 Node.js）
  · 用一个 5.6KB 的 C# 启动器冒充 *.exe，把参数转发给 node app.js
  · 插件包内的 main.js 打了补丁：ENABLE_LOCAL_VIP 由 false 改为 true
    （不打开它，客户端会认为你没开会员，VIP 歌在界面上就被拦下，
      根本不会去取播放地址）

--------------------------------------------------------
 三、包内文件
--------------------------------------------------------

install.ps1                      一键安装脚本
UnblockNeteaseMusic-v0.28.1.exe  C# 启动器（转发参数给 node）
server-src/                      UNM 源码 + precompiled 打包产物 + pino 依赖
server-src/node.exe              便携版 Node.js 24.22.0（无需单独安装）
server-src/launcher.cs           启动器源码（需要时可用 csc 重新编译）
RevivedUnblockNeteaseMusic.plugin 已打补丁的插件包（ENABLE_LOCAL_VIP=true）

安装到目标机器后的位置：
  <BetterNCM数据目录>\RevivedUnblockInstaller\
      UnblockNeteaseMusic-v0.28.1.exe
      server-src\...
      config.json
  <BetterNCM数据目录>\plugins\
      RevivedUnblockNeteaseMusic.plugin
      RevivedUnblockNeteaseMusic.plugin.orig   ← 原包备份

--------------------------------------------------------
 四、常见问题
--------------------------------------------------------

Q: 播出来是一段“酷我音乐为你开启免费听歌权限…”的语音？
A: 说明跑的还是旧版 exe（音源已失效）。检查 <数据目录>\RevivedUnblockInstaller\
   config.json 里 selectedVersion.filename 是否为 UnblockNeteaseMusic-v0.28.1.exe。

Q: 提示无版权 / 需要购买，根本没开始播？
A: 说明 ENABLE_LOCAL_VIP 没生效。检查 plugins\ 下的插件包是否为本包附带的那份
   （BetterNCM 每次启动都会用 .plugin 包覆盖 plugins_runtime，改解压后的文件无效）。

Q: 插件更新后失效？
A: 重新运行本 install.ps1 即可（会重新装回打好补丁的插件包）。

Q: 端口被占用 / 想换端口？
A: 改插件配置 config.json 的 "port"（默认 11724），改完重启网易云。

Q: 音源只有酷我？
A: 酷狗实测全部失败（code=-110），B站/YouTube 也取不到；yt-dlp 源需要自行安装
   且需要非大陆网络，已在配置中关闭。其余音源（QQ/咪咕/JOOX）需要各自 cookie。

Q: 想升级到更新的源码？
A: 需要装 git，然后：
     git clone --depth 1 -b enhanced https://github.com/UnblockNeteaseMusic/server
     用新仓库覆盖 <数据目录>\RevivedUnblockInstaller\server-src\（保留 node.exe）
   注意 precompiled\app.js 才是实际运行的产物，必须一起覆盖；依赖不变时无需重装 pino。
