## 前后端分离
- 基于vue、element-ui构建开发
- 前后端分离，通过token进行数据交互，可独立部署
- npm run dev

# 构建生产环境(默认) npm run build
    # 构建生产环境
    npm run build --prod
    # 安装Nginx，并配置Nginx 
    server {
        listen       80;
        server_name  localhost;
        location / {
            root /home/html/; index index.html index.htm;
        } 
    }
    # 启动Nginx后，访问如下路径即可 http://localhost