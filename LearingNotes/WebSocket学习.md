##9.6
* WebSocket //只有一次握手 

>客户端发送请求切换协议

	GET /chat HTTP/1.1
	Host: server.example.com
	Upgrade: websocket
	Connection: Upgrade
	Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
	Sec-WebSocket-Protocol: chat, superchat
	Sec-WebSocket-Version: 13
	Origin: http://example.com
>服务端收到后返回

	HTTP/1.1 101 Switching Protocols
	Upgrade: websocket
	Connection: Upgrade
	Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
	Sec-WebSocket-Protocol: chat
	Upgrade: websocket
	Connection: Upgrade

* 服务端

		$master = socker_creat(AF_INRT,SOCK_STREAM,SOL_TCP); //创建服务端套接字
		socket_bind($master,'127.0.0.1','8080');//套接字绑定端口
		socket_listen($master，'9') //监听端口，最大连接数为9
		$spawn = socket_accetp($master); //接收客户端，返回客户端套接字
		socket_write($spwan,$str,strlen($str));//向客户端写入信息，需要bulid()先将信息转化成WebSocket数据帧，否则乱码
		$str = socket_read($spwan) //读取客户端发送过来的信息,需要parse()解析，否则乱码
		$butey = socket_recv($spwan,$buffer,2048,0);//同样可以获取客户端发送过来的信息，储存在$buffer，返回的$butey<9则表示套接字已断开（下线）

		function build($msg) {//封装数据
		    $msg = json_encode($msg);
			$frame = [];
			$frame[0] = '81';
			$len = strlen($msg);
			if ($len < 126) {
			    $frame[1] = $len < 16 ? '0' . dechex($len) : dechex($len);
			} else if ($len < 65025) {
			    $s = dechex($len);
			    $frame[1] = '7e' . str_repeat('0', 4 - strlen($s)) . $s;
			} else {
			    $s = dechex($len);
			    $frame[1] = '7f' . str_repeat('0', 16 - strlen($s)) . $s;
			}
			
			$data = '';
			$l = strlen($msg);
			for ($i = 0; $i < $l; $i++) {
			    $data .= dechex(ord($msg{$i}));
			}
			$frame[2] = $data;
			
			$data = implode('', $frame);
			
			return pack("H*", $data);
		}
		
		function parse($buffer) {//解析数据
		    $decoded = '';
		    $len = ord($buffer[1]) & 127;
		    if ($len === 126) {
		        $masks = substr($buffer, 4, 4);
		        $data = substr($buffer, 8);
		    } else if ($len === 127) {
		        $masks = substr($buffer, 10, 4);
		        $data = substr($buffer, 14);
		    } else {
		        $masks = substr($buffer, 2, 4);
		        $data = substr($buffer, 6);
		    }
		    for ($index = 0; $index < strlen($data); $index++) {
		        $decoded .= $data[$index] ^ $masks[$index % 4];
		    }
		    return json_decode($decoded, true);
		}

//请求握手的消息不需要parse()解析

		

	