---
title: "Oauth2 Proxy"
date: 2022-02-18T16:09:35+08:00
description: "oauth2 proxy"
tags: ["oauth2","openresty"]
categories: ["devops"]
keywords: ["oauth2","openresty"]
isCJKLanguage: true
draft: false
---

book.test.io实现oauth鉴权

通过cookie中的自定义cookie TESTAccessToken鉴权，

鉴权失败，跳转登录页面，登录成功把access_token设置到cookie TESTAccessToken


```
opm install ledgetech/lua-resty-http
```

```
#nginx.conf

resolver 114.114.114.114;
lua_ssl_verify_depth 10;
lua_ssl_trusted_certificate '/etc/ssl/certs/ca-certificates.crt';

init_by_lua_block {
    require('oauth2').init({
        code_endpoint = 'https://accounts.test.io/oauth/authorize',
        token_endpoint = 'https://accounts.test.io/oauth/token',
        validate_endpoint = 'https://accounts.test.io/users/info/me',
        redirect_uri = 'https://book.test.io/oauth2/callback',
        start_uri = 'https://book.test.io/',
        client_id = 'client_id',
        client_secret = 'secret',
        scope = 'TEST',
        response_type = 'code',
    })
}

server {
    listen 80;

    location = /validate {
        content_by_lua_block {
            local access_token = ngx.var.cookie_TESTAccessToken
            if access_token then
                local resp_status_code = require('oauth2').validate(access_token)
                ngx.log(ngx.ERR,resp_status_code)
                if  resp_status_code == 200 then
                    ngx.exit(ngx.HTTP_OK)
                else
                    ngx.exit(ngx.HTTP_UNAUTHORIZED)
                end
            else
                ngx.exit(ngx.HTTP_UNAUTHORIZED)
            end

        }
    }

    location @error401 {
        content_by_lua_block {
            return require('oauth2').get_code()
        }
    }

    location / {
        auth_request /validate;
        proxy_pass http://pro-gitbook-service:4000;
        error_page 401 = @error401;
    }

    location /oauth2/callback {
        content_by_lua_block {
        return require('oauth2').set_cookie(ngx.var.arg_code)
        }
    }
}
```

```lua
#oauth.lua

-- resty/lua/oauth2.lua

local cjson = require('cjson.safe')
cjson.encode_empty_table_as_object(false)

local http = require("resty.http")

local M = {}
local _conf = nil

local function code_url()
  local params = ngx.encode_args({
    client_id = _conf.client_id,
    redirect_uri = _conf.redirect_uri,
    scope = _conf.scope,
    response_type = _conf.response_type,
  })
  return _conf.code_endpoint .. '?' .. params
end


local function get_token(code)
    local params = ngx.encode_args({
        code = code,
        client_id = _conf.client_id,
        client_secret = _conf.client_secret,
        grant_type = 'authorization_code',
        redirect_uri = _conf.redirect_uri,
      })
    local httpc = http.new()
    local res, err = httpc:request_uri(_conf.token_endpoint, {
        method = "POST",
        headers = {
            ["Content-Type"] = "application/x-www-form-urlencoded",
        },
        body = params,
    })

    if not res then
        ngx.log(ngx.ERR, "request failed: ", err)
        return
    end

    local status = res.status
    local length = res.headers["Content-Length"]
    local body   = res.body
    return cjson.decode(body)['access_token']
end


function M.validate(token)
    local httpc = http.new()
    local res, err = httpc:request_uri(_conf.validate_endpoint, {
        method = "GET",
        headers = {
            ["Content-Type"] = "application/json",
            ["Authorization"] = 'Bearer ' .. token
        },
    })
    if not res then
        ngx.log(ngx.ERR, "request failed: ", err)
        return 401
    end


    local status = res.status
    local length = res.headers["Content-Length"]
    local body   = res.body
    return status
end

function M.get_code()
  return ngx.redirect(code_url())
end


function M.set_cookie(code)
    local access_token = get_token(code)
    ngx.log(ngx.ERR,access_token)
    ngx.header["Set-Cookie"] = "TESTAccessToken="..access_token.."; path=/;Max-Age=86400"
    ngx.redirect(_conf.start_uri)
end


function M.init(conf)
  _conf = conf
end

return M
```

