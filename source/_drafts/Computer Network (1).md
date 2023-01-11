
Encapsulation（封装）
![Screenshot 2023-01-11 at 10.53.04 AM.png](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/Screenshot%202023-01-11%20at%2010.53.04%20AM.png)

就是每一层的封装后的数据作为下一层的data，data加上headers和footers封装完又成为下一层的data。
![Screenshot 2023-01-11 at 10.59.27 AM.png](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/Screenshot%202023-01-11%20at%2010.59.27%20AM.png)
用Wireshark抓包可以看到，http被tcp包裹，tcp被ip包裹。
![Screenshot 2023-01-11 at 11.03.30 AM.png](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2023-01/Screenshot%202023-01-11%20at%2011.03.30%20AM.png)
对于封装还可以是recursively, 这里讲了vpn的包的构成，和我想象的不一样，我之前以为是和正常的包一样，只不过destinationip变为代理的ip，然后代理解包后在封装发到真正要访问的ip。