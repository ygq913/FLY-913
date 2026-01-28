<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>simrobot connect to GC</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#simrobot连接外部gamecontroller">SimRobot连接外部GameController</a>
<ul>
<li><a href="#目录">目录</a></li>
<li><a href="#概述">概述</a></li>
<li><a href="#更新simulatednao模块">1. 更新SimulatedNao模块</a></li>
<li><a href="#编译项目">2. 编译项目</a></li>
<li><a href="#更新gamecontroller3">3. 更新GameController3</a></li>
<li><a href="#启动gamecontroller3">4. 启动GameController3</a></li>
<li><a href="#启动simrobot">5. 启动SimRobot</a></li>
<li><a href="#验证连接">6. 验证连接</a></li>
<li><a href="#故障排查">7.故障排查</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="simrobot连接外部gamecontroller">SimRobot连接外部GameController</h1>
<h2 id="目录">目录</h2>
<ul>
<li>
<p><a href="#%E6%A6%82%E8%BF%B0">概述</a></p>
</li>
<li>
<p><a href="#1-%E6%9B%B4%E6%96%B0simulatednao%E6%A8%A1%E5%9D%97">1. 更新SimulatedNao模块</a></p>
</li>
<li>
<p><a href="#2-%E7%BC%96%E8%AF%91%E9%A1%B9%E7%9B%AE">2. 编译项目</a></p>
</li>
<li>
<p><a href="#3-%E6%9B%B4%E6%96%B0gamecontroller3">3. 更新GameController3</a></p>
</li>
<li>
<p><a href="#4-%E5%90%AF%E5%8A%A8gamecontroller3">4. 启动GameController3</a></p>
</li>
<li>
<p><a href="#5-%E5%90%AF%E5%8A%A8simrobot">5. 启动SimRobot</a></p>
</li>
<li>
<p><a href="#6-%E9%AA%8C%E8%AF%81%E8%BF%9E%E6%8E%A5">6. 验证连接</a></p>
</li>
<li>
<p><a href="#%E6%95%85%E9%9A%9C%E6%8E%92%E6%9F%A5">7.故障排查</a></p>
</li>
</ul>
<h2 id="概述">概述</h2>
<p>实现通过外部GameController3控制SimRobot仿真环境，替代原有的内部命令控制方式。</p>
<hr>
<h2 id="更新simulatednao模块">1. 更新SimulatedNao模块</h2>
<h3 id="修改的文件">修改的文件</h3>
<p>为了使Simrobot仿真能够连接上GC，对<code>Src/Libs/SimulatedNao</code>文件夹做了以下更改：</p>
<p><strong>新增文件：</strong></p>
<ul>
<li><code>CommandServer.cpp</code> - TCP命令服务器实现</li>
<li><code>CommandServer.h</code> - TCP命令服务器头文件</li>
</ul>
<p><strong>修改文件：</strong></p>
<ul>
<li><code>GameController.cpp</code> - 添加UDP通信支持，实现与外部GC的消息收发</li>
<li><code>GameController.h</code> - 添加UDP套接字和外部控制接口</li>
<li><code>ConsoleRoboCupCtrl.cpp</code> - 添加<code>gc external on/off</code>命令处理</li>
<li><code>ConsoleRoboCupCtrl.h</code> - 集成CommandServer</li>
</ul>
<h3 id="获取更新后的simulatednao">获取更新后的SimulatedNao</h3>
<p><strong>方法1：直接拉取</strong></p>
<pre class=" language-bash"><code class="prism  language-bash">
<span class="token function">cd</span>  ~/2025BHumanCodeRelease/Src/Libs/
<span class="token function">rm</span>  -rf  SimulatedNao
<span class="token function">wget</span>  http://212.64.83.115:5244/ye/SimulatedNao.tar.gz
<span class="token function">tar</span>  -xzf  SimulatedNao.tar.gz
</code></pre>
<p><strong>方法2：Git拉取</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/Src/Libs/
<span class="token function">git</span>  clone  http://212.64.83.115:5244/ye/SimulatedNao.git
</code></pre>
<hr>
<h2 id="编译项目">2. 编译项目</h2>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/Make/Linux/
./compile  Develop
</code></pre>
<hr>
<h2 id="更新gamecontroller3">3. 更新GameController3</h2>
<h3 id="修改的文件-1">修改的文件</h3>
<p>对<code>GameController3</code>，修改了以下文件：</p>
<p><strong>修改文件：</strong></p>
<ul>
<li><code>game_controller_msgs/src/control_message.rs</code> - 修改控制消息格式以适配SimRobot</li>
<li><code>game_controller_app/src/handlers.rs</code> - 添加SimRobot通信处理逻辑</li>
</ul>
<h3 id="获取更新后的gamecontroller3">获取更新后的GameController3</h3>
<p><strong>方法1：直接拉取</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/
<span class="token function">rm</span>  -rf  GameController3
<span class="token function">wget</span>  http://212.64.83.115:5244/ye/GameController3.tar.gz
<span class="token function">tar</span>  -xzf  GameController3.tar.gz
</code></pre>
<p><strong>方法2：Git拉取</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/
<span class="token function">git</span>  clone  http://212.64.83.115:5244/ye/GameController3.git
</code></pre>
<hr>
<h2 id="启动gamecontroller3">4. 启动GameController3</h2>
<h3 id="配置步骤">配置步骤</h3>
<ol>
<li>启动GameController3程序</li>
<li>进行以下配置：</li>
</ol>
<ul>
<li><strong>Competition</strong>: 选择 <code>Champions Cup 5 vs. 5</code></li>
<li><strong>Teams</strong>: 暂时只能选择队伍编号 <code>5</code> 和 <code>70</code></li>
<li><strong>Testing</strong>: 勾选 <code>No Delay</code></li>
<li><strong>Interface</strong>: 选择 <code>lo</code> (本地回环网络)</li>
<li><strong>Casting</strong>: 勾选 <code>Multicast</code> (组播模式)</li>
</ul>
<ol start="3">
<li>点击 <code>Start</code> 开始</li>
</ol>
<hr>
<h2 id="启动simrobot">5. 启动SimRobot</h2>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/Build/Linux/SimRobot/Develop/
./SimRobot
</code></pre>
<h3 id="操作步骤">操作步骤</h3>
<ol>
<li>在SimRobot中打开场景文件（<code>.ros3</code>结尾）</li>
<li>在控制台输入以下命令启用外部GC控制：</li>
</ol>
<pre><code>gc external on
</code></pre>
<hr>
<h2 id="验证连接">6. 验证连接</h2>
<h3 id="检查连接状态">检查连接状态</h3>
<ul>
<li>在GameController3界面观察所有机器人是否显示为已连接状态</li>
<li>SimRobot控制台应显示"External GameController enabled"</li>
</ul>
<h2 id="故障排查">7.故障排查</h2>
<p><strong>问题1：机器人无法连接</strong></p>
<ul>
<li>检查Interface是否设置为<code>lo</code>（本地回环）</li>
<li>确认Multicast已勾选</li>
<li>检查SimRobot控制台是否成功执行<code>gc external on</code></li>
<li>重启SimRobot和GameController3，按顺序重新连接</li>
</ul>
<p><strong>问题2：UDP端口冲突</strong></p>
<ul>
<li>确保端口3838（控制消息）和3939（状态消息）未被占用</li>
<li>使用命令检查：<code>netstat -tulpn | grep -E "3838|3939"</code></li>
</ul>
<p><strong>问题3：消息收发异常</strong></p>
<ul>
<li>检查防火墙设置，确保允许本地UDP通信</li>
<li>查看GameController3日志文件（<code>logs/</code>目录）</li>
<li>查看SimRobot控制台输出的错误信息</li>
</ul>
<p><strong>问题4：队伍编号不匹配</strong></p>
<ul>
<li>确保GameController3中选择的队伍编号与SimRobot场景文</li>
</ul>

    </div>
  </div>
</body>

</html>
