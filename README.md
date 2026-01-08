# Web Unlocker API

[![Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/) 

[Web Unlocker](https://brightdata.co.kr/products/web-unlocker)는 고도화된 봇 보호를 우회하면서 어떤 웹사이트에도 접근할 수 있도록 해주는 강력한 스크レイピング API입니다. 복잡한 アンチボット 인프라를 관리하지 않고도 단일 API 호출로 깔끔한 HTML/JSON レスポンス를 가져올 수 있습니다.

# Table of Contents
- [Features](#features)
- [Getting Started](#getting-started)
   - [Direct API Access](#direct-api-access)
   - [Native Proxy-Based Access](#native-proxy-based-access)
- [Practical Example: Scraping G2 Reviews](#practical-example-scraping-g2-reviews)
   - [Basic Request (Without Web Unlocker)](#basic-request-without-web-unlocker)
   - [Enhanced Request (With Web Unlocker)](#enhanced-request-with-web-unlocker)
      - [Direct API Access](#direct-api-access)
      - [Proxy-Based Access](#proxy-based-access)
      - [Waiting for Specific Elements](#waiting-for-specific-elements)
      - [Mobile User-Agent Targeting](#mobile-user-agent-targeting)
      - [Geolocation Targeting](#geolocation-targeting)
      - [Debugging Requests](#debugging-requests)
      - [Success Rate Statistics](#success-rate-statistics)
- [Final Notes](#final-notes)

## Features
Web Unlocker는 포괄적인 Webスクレイピング 기능을 제공합니다:
- 자동 プロキシ 관리 및 CAPTCHA 해결
- 실제 사용자 행동 시뮬레이션
- 내장 JavaScript 렌더링
- 글로벌 ジオロケーション 타기팅
- 자동 リトライ 메커니즘
- 성공당 과금(Pay-per-success) 요금 모델

## Getting Started
Web Unlocker를 사용하기 전에 [quickstart guide](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart)를 따라 설정을 완료하십시오.

### Direct API Access
Web Unlocker를 통합하는 데 권장되는 방법입니다.


**Example: cURL Command**
```bash
curl -X POST "https://api.brightdata.com/request" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer INSERT_YOUR_API_TOKEN" \
-d '{
  "zone": "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME",
  "url": "http://lumtest.com/myip.json",
  "format": "raw"
}'
```

1. API Endpoint: `https://api.brightdata.com/request`
2. Authorization Header: Web Unlocker API zone의 [API token](https://docs.brightdata.com/scraping-automation/web-unlocker/send-your-first-request#generating-your-bright-data-api-token)
3. Payload:
   - `zone`: 사용 중인 Web Unlocker API zone 이름
   - `url`: 접근할 대상 URL
   - `format`: レスポンス 형식(사이트의 직접 レスポンス를 받으려면 `raw` 사용)

**Example: Python Script**
```python
import requests

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME"
TARGET_URL = "http://lumtest.com/myip.json"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}

payload = {
    "zone": ZONE_NAME,
    "url": TARGET_URL,
    "format": "raw"
}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    print("Success:", response.text)
else:
    print(f"Error {response.status_code}: {response.text}")
```

### Native Proxy-based Access

プロキシ 기반 라우팅을 사용하는 대체 방법입니다.

**Example: cURL Command**
```bash
curl "http://lumtest.com/myip.json" \
--proxy "brd.superproxy.io:33335" \
--proxy-user "brd-customer-<CUSTOMER_ID>-zone-<ZONE_NAME>:<ZONE_PASSWORD>"
```

필수 자격 증명:
1. Customer ID: [Account settings](https://brightdata.co.kr/cp/setting/customer_details)에서 확인합니다.
2. Web Unlocker API zone 이름: overview 탭에서 확인합니다.
3. Web Unlocker API 비밀번호: overview 탭에서 확인합니다.

**Example: Python Script**
```python
import requests

customer_id = "<customer_id>"
zone_name = "<zone_name>"
zone_password = "<zone_password>"

host = "brd.superproxy.io"
port = 33335
proxy_url = f"http://brd-customer-{customer_id}-zone-{zone_name}:{zone_password}@{host}:{port}"

proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get("http://lumtest.com/myip.json", proxies=proxies)

if response.status_code == 200:
    print(response.json())
else:
    print(f"Error: {response.status_code}")
```

## Practical Example: Scraping G2 Reviews
Cloudflare로 강력하게 보호되는 사이트인 [G2.com](https://www.g2.com/)에서 리뷰를 스크レイピング하는 방법을 살펴보겠습니다.

### Basic Request (Without Web Unlocker)
간단한 Python 스크립트를 사용하여 [G2 reviews](https://www.g2.com/products/mongodb/reviews)를 스크レイピング합니다:
```python
import requests
from bs4 import BeautifulSoup

url = 'https://www.g2.com/products/mongodb/reviews'
response = requests.get(url)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = soup.find_all('h2')
    
    if headings:
        print("\nHeadings Found:")
        for heading in headings:
            print(f"- {heading.get_text(strip=True)}")
    else:
        print("No headings found")
else:
    print("Request blocked")
```

**Result:** Cloudflare의 アンチボット 조치로 인해 스크립트가 실패합니다(`403` 오류).


### Enhanced Request (With Web Unlocker)
이러한 제한을 우회하려면 Web Unlocker를 사용하십시오. 아래는 Python 구현 예시입니다:

#### Direct API Access
```python
import requests
from bs4 import BeautifulSoup

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_ZONE"
TARGET_URL = "https://www.g2.com/products/mongodb/reviews"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}
payload = {"zone": ZONE_NAME, "url": TARGET_URL, "format": "raw"}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```
**Result:** 보호를 성공적으로 우회하고 상태 `200`으로 콘텐츠를 가져옵니다.

#### Proxy-Based Access
대안으로 プロキシ 기반 방법을 사용할 수 있습니다:
```python
import requests
from bs4 import BeautifulSoup

proxy_url = "http://brd-customer-<customer_id>-zone-<zone_name>:<zone_password>@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

url = "https://www.g2.com/products/mongodb/reviews"
response = requests.get(url, proxies=proxies, verify=False)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```

**Note:** 다음을 추가하여 SSL 인증서 경고를 억제하십시오:
```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

#### Waiting for Specific Elements
`x-unblock-expect` ヘッダー를 사용하여 특정 요소 또는 텍스트를 기다릴 수 있습니다:
```python
headers["x-unblock-expect"] = '{"element": ".star-wrapper__desc"}'
# or
headers["x-unblock-expect"] = '{"text": "reviews"}'
```

👉 전체 코드는 [g2_wait.py](https://github.com/bright-kr/web-unlocker/blob/main/src/g2_wait.py)에서 확인할 수 있습니다.

#### Mobile User-Agent Targeting
데스크톱 user agent 대신 모바일 user agent를 사용하려면 username에 `-ua-mobile`을 추가하십시오:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-ua-mobile"
```
👉 전체 코드는 [g2_mobile.py](https://github.com/bright-kr/web-unlocker/blob/main/src/g2_mobile.py)에서 확인할 수 있습니다.

#### Geolocation Targeting
Web Unlocker는 자동으로 최적의 IP 위치를 선택하지만, 대상 위치를 지정할 수도 있습니다:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us"
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us-city-sanfrancisco"
```

👉 자세한 내용은 [here](https://docs.brightdata.com/api-reference/proxy/geolocation-targeting)에서 확인할 수 있습니다.

#### Debugging Requests
`-debug-full` 플래그를 추가하여 상세 디버깅 정보를 활성화하십시오:
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-debug-full"
```
👉 전체 코드는 [g2_debug.py](https://github.com/bright-kr/web-unlocker/blob/main/src/g2_debug.py)에서 확인할 수 있습니다.

#### Success Rate Statistics
특정 도메인에 대한 API 성공률을 모니터링합니다:
```python
import requests

API_TOKEN = "INSERT_YOUR_API_TOKEN"

def get_success_rate(domain):
    url = f"https://api.brightdata.com/unblocker/success_rate/{domain}"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {API_TOKEN}"
    }
    response = requests.get(url, headers=headers)
    print(response.json() if response.status_code == 200 else response.text)

get_success_rate("g2.com") # Get statistics for specific domain
get_success_rate("g2.*") # Get statistics for all top-level domains
```

## Final Notes
Web Unlocker를 사용하면 가장 강력하게 보호되는 웹사이트도 손쉽게 스크レイピング할 수 있습니다. 기억해야 할 핵심 사항은 다음과 같습니다:

1. **Not Compatible With**:  
   - 브라우저(Chrome, Firefox, Edge)  
   - 안티-디텍트 브라우저(Adspower, Multilogin)  
   - 자동화 도구(Puppeteer, Playwright, Selenium)  

2. **Use Scraping Browser**:  
   브라우저 기반 자동화에는 Bright Data의 [Scraping Browser](https://brightdata.co.kr/products/scraping-browser)를 사용하십시오.

3. **Premium Domains**:  
   [premium domain](https://docs.brightdata.com/scraping-automation/web-unlocker/features#web-unlocker-api-premium-domains) 기능을 통해 난이도 높은 사이트에 접근할 수 있습니다.

4. **CAPTCHA Solving**:  
   자동으로 해결되지만 [disabled](https://docs.brightdata.com/scraping-automation/web-unlocker/features#disable-captcha-solving)할 수 있습니다. Bright Data의 [CAPTCHA Solver](https://brightdata.co.kr/products/web-unlocker/captcha-solver)에 대해 더 알아보십시오.
   
5. **Custom Headers & Cookies**:  
   특정 사이트 버전을 타기팅하기 위해 사용자 지정 ヘッダー 및 Cookie를 전송할 수 있습니다. [Learn more](https://docs.brightdata.com/scraping-automation/web-unlocker/features#manual-headers-and-cookies).

자세한 내용은 [official documentation](https://docs.brightdata.com/scraping-automation/web-unlocker/introduction)을 방문하여 확인하십시오.