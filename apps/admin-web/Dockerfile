FROM node:24.18.0-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN test "$(node --version)" = "v24.18.0" \
    && test "$(npm --version)" = "11.16.0" \
    && npm ci --no-audit --no-fund --strict-allow-scripts
COPY . .
# 构建期环境变量：.env* 既不在版本库里（.gitignore）也被 .dockerignore 排除，不显式提供时 import.meta.env 为空，
# axios 没有 baseURL，所有接口请求都会落到 nginx 的静态回退上（登录直接 405）。Vite 会读取进程环境里 VITE_ 前缀的变量。
# VITE_ACCESS_MODE 必须是 frontend：后端动态菜单接口 /admin/menus 固定返回 503（"尚未启用，请使用前端路由模式"）。
ARG VITE_API_URL=/admin-api
ARG VITE_ACCESS_MODE=frontend
ARG VITE_WITH_CREDENTIALS=false
ARG VITE_OPEN_ROUTE_INFO=false
# 前台（user-web）地址，管理端"打开前台"功能使用；留空时该功能会提示未配置，按部署通过 build args 传入
ARG VITE_USER_WEB_URL=
ENV VITE_API_URL=$VITE_API_URL \
    VITE_ACCESS_MODE=$VITE_ACCESS_MODE \
    VITE_WITH_CREDENTIALS=$VITE_WITH_CREDENTIALS \
    VITE_OPEN_ROUTE_INFO=$VITE_OPEN_ROUTE_INFO \
    VITE_USER_WEB_URL=$VITE_USER_WEB_URL
RUN npm run typecheck && npm run build

FROM nginxinc/nginx-unprivileged:1.30.3-alpine
COPY --from=build --chown=101:101 /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
