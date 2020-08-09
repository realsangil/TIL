# mitmproxy 설치하기

## mitmproxy 이란?
> mitmproxy is a free and open source interactive HTTPS proxy.

간단하게 설명하자면 클라이언트와 서버간의 HTTP(S) Request와 Response를 모니터링하고 Debugging 할 수 있는 오픈소스 툴입니다.

여기서 mitm의 의미를 짚고 넘어가자면 mitm은 `man in the middle`의 약자이며 직역하면 `중간자`라고 할 수 있습니다.

![그림1](imagesreq-res.jpg)

그림1은 중간자가 없는 기존 클라이언트와 서버와의 통신 구조는 간략하게 표현한 것입니다.

![그림2](imagesreq-res-with-mitm.jpg)

그림2는 중간자가 포함된 클라이언트와 서버의 통신 구조입니다.

그림2와 같이 중간자가 클라이언트와 서버 사이에 위치하기 때문에 Request와 Response를 모니터링할 수 있게 됩니다.

mitmproxy가 동작하는 자세한 원리는 공식 문서에 [How mitmproxy works](https://docs.mitmproxy.org/stable/concepts-howmitmproxyworks/)에서 확인 가능합니다.

## 설치

설치 과정은 macOS를 기준으로 설명하겠습니다.

### mitmproxy 설치 후 실행
```bash
# homebrew를 통한 설치
brew install mitmproxy

# mitmproxy 실행
mitmproxy
```
Command를 실행하면 기본 포트인 `8080`로 실행된 것을 알 수 있습니다.
mitmproxy를 실행하면 `$Home/.mitmproxy` 디렉토리와  인증서들을 생성합니다.

### mitmproxy 인증서 설치
```bash
# 인증서 등록
sudo security add-trusted-cert -d -r trustRoot -p ssl -k /Library/Keychains/System.keychain $HOME/.mitmproxy/mitmproxy-ca-cert.pem 
```

Command를 실행하면 시스템 키체인에 mitmproxy 인증서를 등록합니다.

![인증서 등록](images/certificates-registration.png)

### proxy 설정
```bash
# 프록시 설정
networksetup -setwebproxy "Wi-Fi" localhost 8080
networksetup -setsecurewebproxy "Wi-Fi" localhost 8080
```

Command를 실행하면 `WIFI` 인터페이스의 프록시를 `127.0.0.1:8080`로 세팅합니다.

확인은 `설정-네트워크-WIFI-고급`에서 확인할 수 있습니다.

![proxy 설정](images/proxy-setting.png)

## 삭제
```bash
# mitmproxy 삭제
brew uninstall mitmproxy
# 인증서 삭제
sudo security delete-certificate -c mitmproxy
# 프록시 설정 끄기
networksetup -setwebproxystate "Wi-Fi" off
networksetup -setsecurewebproxystate "Wi-Fi" off
```

삭제는 설치의 역순이지만 위 Command를 이용하면 더욱 간단합니다.

## 실행과 종료

기본적으로 fiddler나 charles 같은 툴들은 프로그램을 실행할 때 proxy를 세팅해주고 종료될 때 다시 복원해주는 작업을 자동적으로 해주지만 mitmproxy에 그런 기능은 없기 때문에 실행할 때 proxy를 세팅해주고 종료할 땐 반드시 proxy 세팅을 복원해야 합니다.

### 실행 
```bash
# 프록시 설정
networksetup -setwebproxy 'Wi-Fi' localhost 8080 
networksetup -setsecurewebproxy 'Wi-Fi' localhost 8080

# mitmproxy 실행
mitmproxy
```

### 종료
```bash
# mitmproxy 종료 후 proxy 설정 복원
networksetup -setwebproxystate "Wi-Fi" off
networksetup -setsecurewebproxystate "Wi-Fi" off
```

## 테스트

![인증서 확인](imagescertificates.png)

네이버로 접속 후 인증서를 확인하면 발급자가 mitmproxy로 변경된 것을 확인할 수 있으며 mitmproxy에서는 network flow를 확인할 수 있습니다.

![네트워크 흐름 확인](imagesnetwork-flow.png)

## 정리

이번 포스트는 mitmproxy를 설치하고 설정하는 방법에 대해서 정리해보았습니다.

mitmproxy는 TUI뿐만 아니라 `mitmweb`이라는 브라우저 기반의 GUI로도 이용하실 수 있습니다.

mitmproxy를 사용해보고자 하는 분들에게 조금이나마 도움이 되었기를 바라면서 이번 포스팅을 마치겠습니다.

궁금하신 점이나 수정 보완해야할 점들은 댓글이나 [psi59@lalaworks.com](mailto:psi59@lalaworks.com)으로 메일 주시길 바랍니다.



