<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GC添加暂停键按钮</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#gamecontroller3-暂停按钮功能说明">GameController3 暂停按钮功能说明</a>
<ul>
<li><a href="#目录">目录</a></li>
<li><a href="#概述">概述</a></li>
<li><a href="#版本说明">1. 版本说明</a></li>
<li><a href="#更新内容">2. 更新内容</a></li>
<li><a href="#编译运行">3. 编译运行</a></li>
<li><a href="#功能使用">4. 功能使用</a></li>
<li><a href="#验证测试">5. 验证测试</a></li>
<li><a href="#附录">6.附录</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="gamecontroller3-暂停按钮功能说明">GameController3 暂停按钮功能说明</h1>
<h2 id="目录">目录</h2>
<ul>
<li><a href="#%E6%A6%82%E8%BF%B0">概述</a></li>
<li><a href="#1-%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E">1. 版本说明</a></li>
<li><a href="#2-%E6%9B%B4%E6%96%B0%E5%86%85%E5%AE%B9">2. 更新内容</a></li>
<li><a href="#3-%E7%BC%96%E8%AF%91%E8%BF%90%E8%A1%8C">3. 编译运行</a></li>
<li><a href="#4-%E5%8A%9F%E8%83%BD%E4%BD%BF%E7%94%A8">4. 功能使用</a></li>
<li><a href="#5-%E9%AA%8C%E8%AF%81%E6%B5%8B%E8%AF%95">5. 验证测试</a></li>
<li><a href="#5-%E9%AA%8C%E8%AF%81%E6%B5%8B%E8%AF%95">6. 附录</a></li>
</ul>
<hr>
<h2 id="概述">概述</h2>
<p>本版本在 <code>GameController3 （gc与simrobot连接）</code> 的基础上添加了<code>全部暂停</code>功能，允许裁判在比赛过程中暂停所有机器人的动作。<br>
环境依赖编译：请参考<code>GC3拉取及编译.pdf</code>文件的第一点</p>
<hr>
<h2 id="版本说明">1. 版本说明</h2>
<p><strong>基础版本：</strong><code>GameController3</code><br>
<strong>第二版本：</strong>  <code>GameController3 （simrobot连接）</code><br>
<strong>当前版本：</strong>  <code>GameController3暂停按键</code><br>
<strong>主要功能：</strong> 添加暂停/恢复按钮，支持一键暂停所有机器人</p>
<hr>
<h2 id="更新内容">2. 更新内容</h2>
<h3 id="后端修改">2.1后端修改</h3>
<p><strong>新增文件：</strong></p>
<ul>
<li><code>game_controller_core/src/actions/pause.rs</code> - 暂停动作实现，处理暂停状态切换</li>
<li><code>game_controller_core/src/actions/timeout.rs</code> - 超时状态处理，管理暂停计时</li>
</ul>
<p><strong>修改文件：</strong></p>
<ul>
<li><code>game_controller_core/src/actions/mod.rs</code> - 注册暂停和恢复动作</li>
<li><code>game_controller_msgs/src/control_message.rs</code> - 添加暂停状态的网络消息映射</li>
</ul>
<h3 id="前端修改">2.2前端修改</h3>
<p><strong>新增文件：</strong></p>
<ul>
<li><code>frontend/src/components/main/PauseAllButton.jsx</code> - 暂停按钮UI组件</li>
</ul>
<p><strong>修改文件：</strong></p>
<ul>
<li><code>frontend/src/components/main/StatePanel.jsx</code> - 在状态面板中集成暂停按钮</li>
<li><code>frontend/src/actions.js</code> - 添加暂停和恢复动作定义</li>
</ul>
<h3 id="配置修改">2.3配置修改</h3>
<p><strong>修改文件：</strong></p>
<ul>
<li><code>game_controller_app/tauri.conf.json</code> - 移除窗口配置冲突，支持窗口大小调整</li>
</ul>
<hr>
<h2 id="编译运行">3. 编译运行</h2>
<h3 id="编译步骤">3.1编译步骤</h3>
<p><strong>1. 编译前端：</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/GameController3暂停按键/frontend
<span class="token function">npm</span>  <span class="token function">install</span>
<span class="token function">npm</span>  run  build
</code></pre>
<p><strong>2. 编译后端：</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/GameController3暂停按键
cargo  build  --release
</code></pre>
<h3 id="运行步骤">3.2运行步骤</h3>
<p><strong>启动GameController：</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span>  ~/2025BHumanCodeRelease/GameController3暂停按键/target/release
./game_controller_app
</code></pre>
<p><strong>配置参数：</strong></p>
<ul>
<li><strong>Competition</strong>: 选择 <code>Champions Cup 5 vs. 5</code></li>
<li><strong>Teams</strong>: 暂时只能选择队伍编号 <code>5</code> 和 <code>70</code></li>
<li><strong>Testing</strong>: 勾选 <code>No Delay</code></li>
<li><strong>Interface</strong>: 选择 <code>lo</code> (本地回环网络)</li>
<li><strong>Casting</strong>: 勾选 <code>Multicast</code> (组播模式)</li>
</ul>
<p>点击 <code>Start</code> 启动比赛控制</p>
<hr>
<h2 id="功能使用">4. 功能使用</h2>
<h3 id="暂停操作">4.1暂停操作</h3>
<p><strong>触发方式：</strong></p>
<ol>
<li>在GameController主界面找到 <code>⏸️ 全部暂停 / PAUSE ALL</code> 按钮</li>
<li>点击按钮执行暂停</li>
</ol>
<p><strong>效果：</strong></p>
<ul>
<li>游戏状态切换为 <code>Timeout</code></li>
<li>所有机器人接收到暂停信号</li>
<li>机器人停止当前动作并保持站立</li>
<li>比赛计时器暂停</li>
</ul>
<h3 id="恢复操作">4.2恢复操作</h3>
<p><strong>触发方式：</strong></p>
<ol>
<li>暂停状态下，按钮显示为 <code>▶️ 恢复比赛 / RESUME</code></li>
<li>点击按钮恢复比赛</li>
</ol>
<p><strong>效果：</strong></p>
<ul>
<li>游戏状态恢复到暂停前的状态</li>
<li>机器人恢复正常比赛行为</li>
<li>比赛计时器继续计时</li>
</ul>
<hr>
<h2 id="验证测试">5. 验证测试</h2>
<h3 id="测试步骤">5.1测试步骤</h3>
<p><strong>1. 启动测试环境：</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 启动SimRobot</span>
<span class="token function">cd</span>  ~/2025BHumanCodeRelease/Build/Linux/SimRobot/Develop/
./SimRobot
<span class="token comment"># 在SimRobot控制台启用外部GC</span>
gc  external  on
</code></pre>
<p><strong>2. 测试暂停功能：</strong></p>
<ul>
<li>在GameController中将比赛状态设置为 <code>Playing</code></li>
<li>观察机器人开始移动</li>
<li>点击"全部暂停"按钮</li>
<li><strong>预期结果</strong>：所有机器人立即停止移动并保持站立</li>
</ul>
<p><strong>3. 测试恢复功能：</strong></p>
<ul>
<li>在暂停状态下点击"恢复比赛"按钮</li>
<li><strong>预期结果</strong>：机器人恢复正常比赛行为</li>
</ul>
<h3 id="检查要点">5.2检查要点</h3>
<p><strong>GameController端：</strong></p>
<ul>
<li>按钮状态正确切换（暂停 ↔ 恢复）</li>
<li>游戏状态显示为 <code>Timeout</code></li>
<li>计时器停止计时</li>
</ul>
<p><strong>机器人端：</strong></p>
<ul>
<li>接收到 <code>GAME_PHASE_TIMEOUT</code> 消息</li>
<li>状态转换为 <code>GameState::timeout</code></li>
<li>执行 <code>Stand</code> 技能（站立不动）</li>
</ul>
<h3 id="故障排查">5.3故障排查</h3>
<p><strong>问题1：点击按钮无反应</strong></p>
<ul>
<li>检查GameController与机器人的网络连接</li>
<li>确认Interface设置为正确的网络接口</li>
<li>查看GameController日志文件（<code>logs/</code>目录）</li>
</ul>
<p><strong>问题2：机器人未停止</strong></p>
<ul>
<li>确认机器人已连接到GameController</li>
<li>检查机器人端是否正确接收状态消息</li>
<li>在SimRobot中查看 <code>theGameState</code> 确认状态</li>
</ul>
<p><strong>问题3：窗口无法调整大小</strong></p>
<ul>
<li>确认已使用最新编译的版本</li>
<li>清除应用缓存：<code>rm -rf ~/.local/share/org.robocup.spl.game-controller</code></li>
<li>重新编译并运行</li>
</ul>
<hr>
<h2 id="附录">6.附录</h2>
<h3 id="相关文件路径">6.1相关文件路径</h3>
<p><strong>GameController端：</strong></p>
<ul>
<li>暂停动作实现：<code>game_controller_core/src/actions/pause.rs</code></li>
<li>超时处理：<code>game_controller_core/src/actions/timeout.rs</code></li>
<li>消息映射：<code>game_controller_msgs/src/control_message.rs</code></li>
<li>UI按钮：<code>frontend/src/components/main/PauseAllButton.jsx</code></li>
</ul>
<p><strong>机器人端：</strong></p>
<ul>
<li>状态转换：<code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code></li>
<li>行为处理：<code>Src/Modules/BehaviorControl/SkillBehaviorControl/Options/HandleRefereeSignal.cpp</code></li>
<li>状态定义：<code>Src/Representations/Infrastructure/GameState.h</code></li>
</ul>
<h3 id="技术说明">6.2技术说明</h3>
<p><strong>状态流转：</strong></p>
<pre><code>
用户点击暂停按钮
	↓
GameController: State::Timeout
	↓ (UDP: GAME_PHASE_TIMEOUT)
机器人: GameState::timeout
	↓
机器人行为: Stand（站立）
</code></pre>
<p><strong>网络协议：</strong></p>
<ul>
<li>使用UDP协议发送控制消息</li>
<li>端口：3838（控制消息）、3939（状态消息）</li>
<li>消息格式：<code>RoboCupGameControlData</code> 结构体</li>
</ul>

    </div>
  </div>
</body>

</html>
