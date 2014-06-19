---
layout: posts
title: "How to download files with WebDriver"
description: "Download files using httpclient in WebDriver"
category: codes
tags: [httpclient, webdriver, automation]
---

```java

package com.test;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Set;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.protocol.ClientContext;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.cookie.BasicClientCookie;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.http.protocol.HttpContext;
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;

/**
 * 
 * @author rissen
 * 
*/
public class HttpClientDownloader {
	private String downloadFileFolder = "c:\\selenium\\";
		private WebDriver webDriver;
		public HttpClientDownloader(WebDriver webDriver) {
			this.webDriver = webDriver;
		}

		public String downloadFileByUrl(String url, String fileNameWithFormat)
				throws IOException {
			HttpClient httpClient = new DefaultHttpClient();

			HttpContext localContext = new BasicHttpContext();
			localContext.setAttribute(ClientContext.COOKIE_STORE,
					getBasicCookieStore(webDriver));

			HttpGet httpGet = new HttpGet(url);
			String downloadFilePath = downloadFileFolder + fileNameWithFormat;

			HttpResponse httpResponse = httpClient.execute(httpGet, localContext);
			int statusCode = httpResponse.getStatusLine().getStatusCode();
			if (statusCode == 200) {
				HttpEntity entity = httpResponse.getEntity();
				FileOutputStream fos = new FileOutputStream(downloadFilePath);
				entity.writeTo(fos);
				fos.close();
				System.out.println("Download file path is " + downloadFilePath);
			} else {
				System.out.println("Download " + fileNameWithFormat
						+ " failed, http response code is " + statusCode);
				throw new IOException("Download file failed, HTTP response code "
						+ statusCode + " - "
						+ httpResponse.getStatusLine().getReasonPhrase());
			}

			return downloadFilePath;
		}

		/**
		 * Initialize http client cookie.
		 * 
		 * @return BasicClientCookie
		 */
		public BasicCookieStore getBasicCookieStore(WebDriver webDriver) {
			Set<Cookie> cookieSet = webDriver.manage().getCookies();
			BasicCookieStore basicCookieStore = new BasicCookieStore();
			for (Cookie ck : cookieSet) {
				BasicClientCookie basicClientCookie = new BasicClientCookie(
						ck.getName(), ck.getValue());
				basicClientCookie.setDomain(ck.getDomain());
				basicClientCookie.setSecure(ck.isSecure());
				basicClientCookie.setExpiryDate(ck.getExpiry());
				basicClientCookie.setPath(ck.getPath());
				basicCookieStore.addCookie(basicClientCookie);
			}
			return basicCookieStore;
		}

		public String getDownloadFileFolder() {
			return downloadFileFolder;
		}

		public void setDownloadFileFolder(String downloadFileFolder) {
			this.downloadFileFolder = downloadFileFolder;
		}

		public WebDriver getWebDriver() {
			return webDriver;
		}

		public void setWebDriver(WebDriver webDriver) {
			this.webDriver = webDriver;
		}
	}
	
```
