#### Client (send)
```python
    import cv2
    import socket

    cap = cv2.VideoCapture(0)
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(('ipaddress', port))
    while cap.isOpened():
        ret, image = cap.read()
        if not ret: break
        client.sendall(image.tostring())
        ...
```

#### Server (receive)
```python
    import socket
    import pickle
    import numpy as np

    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind(('ip address', port))
    server.listen(5)
    conn, addr = server.accept()
    while loop:
        data = b''
        while True:
            buf = conn.recv(1024)
            data += buf
            if len(buf) < 1024:
                break   # EOF
        image = np.fromstring(buf, np.dtype('uint8')).reshape((240, 320, 3))
        ...
```
