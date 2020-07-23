# Unix Socket 테스트

테스팅을 위해 `socat` 툴을 설치

```bash
apt-get install -y socat
```

## Listen to socket
```bash
socat UNIX-CONNECT:/path/to/test.sock -
```

위 커맨드는 `test.sock` 새로 만들거나 이미 존재하는 소켓에 Listen하는 커맨드다.

## Send message to socket
```bash
echo 'hello, world' | socat - UNIX-CONNECT:/path/to/test.sock
```

위 커맨드는 `test.sock`으로 `hello, world`라는 메시지를 보내는 커맨드다.