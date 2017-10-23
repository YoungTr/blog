---
title: 工作记录
date: 2017-10-16 17:34:22
tags: [工作记录]
categories: [工作记录]
---

### 1. [android]在 Html.fromHtml 中的换行被忽略

使用"&lt;br>"标签

```
<font size="3" color="black">" + message + "</font><br>
```

### 2. Android使用HttpURLConnection编程遇到的302重定向问题

接管重定向过程，直接调用HttpURLConnection的setInstanceFollowRedirects(false)，传入参数为false，然后递归的方式进行http请求，然后从header参数中取出location字段。

[获取url重定向之后的地址](http://vjson.com/wordpress/android%E8%8E%B7%E5%8F%96url%E9%87%8D%E5%AE%9A%E5%90%91%E4%B9%8B%E5%90%8E%E7%9A%84%E5%9C%B0%E5%9D%80.html "获取url重定向之后的地址")

```
	public String recursiveTracePath(String path, String reffer) {
		String realURL = null;
		if (mStackDeep.getAndDecrement() > 0) {// 避免异常递归导致StackOverflowError
			URL url = null;
			HttpURLConnection conn = null;
			try {
				url = new URL(path);
				conn = (HttpURLConnection) url.openConnection();
 
				conn.setRequestProperty("User-Agent", DEFAULT_USER_AGENT_PHONE);
				conn.setConnectTimeout(30000);
				conn.setReadTimeout(30000);
				conn.setInstanceFollowRedirects(false);
				int code = conn.getResponseCode();// network block
				if (needRedirect(code)) {
          //临时重定向和永久重定向location的大小写有区分
					String location = conn.getHeaderField("Location");
					if (location == null) {
						location = conn.getHeaderField("location");
					}
 
					if (!(location.startsWith("http://") || location
							.startsWith("https://"))) {
          //某些时候会省略host，只返回后面的path，所以需要补全url
						URL origionUrl = new URL(path);
						location = origionUrl.getProtocol() + "://"
								+ origionUrl.getHost() + location;
					}
 
					return recursiveTracePath(location, path);
				} else {
					// redirect finish.
					realURL = path;
				}
 
			} catch (MalformedURLException e) {
				Log.w(TAG, "recursiveTracePath MalformedURLException");
			} catch (IOException e) {
				Log.w(TAG, "recursiveTracePath IOException");
			} catch (Exception e) {
				Log.w(TAG, "unknow exception");
			} finally {
				if (conn != null) {
					conn.disconnect();
				}
			}
		}
 
		return realURL;
	}

	private boolean needRedirect(int code) {
		return (code == HttpStatus.SC_MOVED_PERMANENTLY
				|| code == HttpStatus.SC_MOVED_TEMPORARILY
				|| code == HttpStatus.SC_SEE_OTHER || code == HttpStatus.SC_TEMPORARY_REDIRECT);
	}
```