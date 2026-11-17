FROM node:24.18.0-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN test "$(node --version)" = "v24.18.0" \
    && test "$(npm --version)" = "11.16.0" \
    && npm ci --no-audit --no-fund --strict-allow-scripts
COPY . .
# 构建期环境变量（原因同 admin-web/Dockerfile：.env* 不在版本库里且被 .dockerignore 排除）。
# 登录/注册要求配置用户协议与隐私政策链接，缺省时登录按钮与协议勾选框会被永久禁用。
# 默认指向 public/legal/ 下的占位页，部署方需替换为正式文本（覆盖这两个文件，或通过 build args 改成自己的 URL）。
ARG VITE_TERMS_URL=/legal/terms.html
ARG VITE_PRIVACY_URL=/legal/privacy.html
ENV VITE_TERMS_URL=$VITE_TERMS_URL \
    VITE_PRIVACY_URL=$VITE_PRIVACY_URL
RUN npm run build

FROM nginxinc/nginx-unprivileged:1.30.3-alpine
COPY --from=build --chown=101:101 /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
