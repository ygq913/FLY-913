<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>口哨检测set-playing状态转换</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li><a href="#口哨检测与-set-→-playing-状态转换指南">口哨检测与 Set → Playing 状态转换指南</a>
<ul>
<li><a href="#目录">目录</a></li>
<li><a href="#功能概述">1. 功能概述</a></li>
<li><a href="#关键参数配置（官方给出的默认值）">2. 关键参数配置（官方给出的默认值）</a></li>
<li><a href="#问题：为什么吹口哨没反应？">3. 问题：为什么吹口哨没反应？</a></li>
<li><a href="#关键代码位置">4. 关键代码位置</a></li>
<li><a href="#simrobot-中测试口哨检测">5. SimRobot 中测试口哨检测</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h1 id="口哨检测与-set-→-playing-状态转换指南">口哨检测与 Set → Playing 状态转换指南</h1>
<h2 id="目录">目录</h2>
<ol>
<li><a href="#1-%E5%8A%9F%E8%83%BD%E6%A6%82%E8%BF%B0">功能概述</a></li>
<li><a href="#2-%E5%85%B3%E9%94%AE%E5%8F%82%E6%95%B0%E9%85%8D%E7%BD%AE%EF%BC%88%E5%AE%98%E6%96%B9%E7%BB%99%E5%87%BA%E7%9A%84%E9%BB%98%E8%AE%A4%E5%80%BC%EF%BC%89">关键参数配置（官方给出的默认值）</a></li>
<li><a href="#3-%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E5%90%B9%E5%8F%A3%E5%93%A8%E6%B2%A1%E5%8F%8D%E5%BA%94%EF%BC%9F">问题：为什么吹口哨没反应？</a></li>
<li><a href="#4-%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%81%E4%BD%8D%E7%BD%AE">关键代码位置</a></li>
<li><a href="#6-%E4%B8%89%E7%A7%8D%E5%87%BA%E7%95%8C%E7%8A%B6%E6%80%81%E8%AF%A6%E8%A7%A3">SimRobot 中测试口哨检测</a></li>
</ol>
<h2 id="功能概述">1. 功能概述</h2>
<p>在 RoboCup SPL 比赛中，机器人需要通过口哨声来触发从 Set 状态到 Playing 状态的转换。B-Human 代码支持两种方式触发这个转换：</p>
<ul>
<li><strong>GameController 发送 PLAYING 指令</strong></li>
<li><strong>机器人检测到口哨声</strong></li>
</ul>
<p>两种方式可以同时生效，代码注释明确写道：</p>
<blockquote>
<p>“Whistle detection for waitFor* states - always active regardless of GameController status. This allows both whistle and GameController to trigger the transition to playing”</p>
</blockquote>
<h2 id="关键参数配置（官方给出的默认值）">2. 关键参数配置（官方给出的默认值）</h2>
<p><strong>修改后的参数见3.5节</strong></p>
<p>配置文件：<code>Config/Scenarios/Default/gameStateProvider.cfg</code></p>

<table>
<thead>
<tr>
<th>参数</th>
<th>默认值</th>
<th>含义</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>minVotersForWhistle</code></td>
<td>5</td>
<td>**己方队伍（一支队伍）**最少需要多少台机器人参与投票</td>
</tr>
<tr>
<td><code>minWhistleAverageConfidence</code></td>
<td>1.15</td>
<td>平均置信度阈值，需要 &gt; 此值才认为检测到口哨</td>
</tr>
<tr>
<td><code>maxWhistleTimeDifference</code></td>
<td>1000ms</td>
<td>多台机器人检测到口哨的时间窗口</td>
</tr>
<tr>
<td><code>ignoreWhistleAfterKickOff</code></td>
<td>3500ms</td>
<td>开球后忽略口哨的时间（避免误检）</td>
</tr>
</tbody>
</table><h2 id="问题：为什么吹口哨没反应？">3. 问题：为什么吹口哨没反应？</h2>
<h3 id="问题现象">3.1 问题现象</h3>
<p><strong>SimRobot 7v7 测试时：</strong> 口哨检测正常工作，观察到的数据：</p>
<ul>
<li><code>confidenceOfLastWhistleDetection = 2</code></li>
<li><code>channelsUsedForWhistleDetection = 2 或 4</code></li>
<li><code>lastTimeWhistleDetected</code> 显示时间戳，正常更新</li>
</ul>
<p>但是我们<strong>日常训练3v3时候的状况</strong>：（因为机器人个数不够，只能3vs3,笑死）吹口哨后机器人没有从 Set 状态进入 Playing 状态。</p>
<p>这说明口哨检测本身是工作的，问题可能出在 <strong>投票人数不足</strong> 导致的<strong>稀释</strong>机制。</p>
<h3 id="稀释置信度是什么意思？">3.2 "稀释置信度"是什么意思？</h3>
<p><strong>核心问题：代码第 598-602 行的逻辑。详细代码见第4部分</strong></p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">if</span><span class="token punctuation">(</span>data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> minVotersForWhistle <span class="token operator">&amp;&amp;</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> numOfVoters<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">const</span>  size_t missing <span class="token operator">=</span> minVotersForWhistle <span class="token operator">-</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	numOfChannels <span class="token operator">+</span><span class="token operator">=</span> <span class="token keyword">static_cast</span><span class="token operator">&lt;</span><span class="token keyword">int</span><span class="token operator">&gt;</span><span class="token punctuation">(</span>missing <span class="token operator">*</span> numOfMissingChannels <span class="token operator">/</span> <span class="token punctuation">(</span>numOfVoters <span class="token operator">-</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>这段代码的意思是：</strong></p>
<ul>
<li>如果听到口哨的机器人数量 &lt; 5（minVotersForWhistle）</li>
<li>并且听到口哨的机器人数量 &lt; 场上总机器人数</li>
<li>那么，把"没听到口哨的机器人的通道数"加到分母 <code>numOfChannels</code> 上</li>
<li>满足上边的情况就会<strong>稀释</strong></li>
</ul>
<p><strong>举个例子说明</strong><br>
因为<strong>minVotersForWhistle默认值</strong>是5，假设 3v3 训练，3 台机器人都听到了口哨(也可能3台没有全部都听到)：</p>
<ul>
<li>每台置信度 = 2，通道数 = 2</li>
<li><code>totalConfidence = 2×2 + 2×2 + 2×2 = 12</code></li>
<li><code>numOfChannels = 2 + 2 + 2 = 6</code></li>
</ul>
<p><strong>代码会稀释！因为 data.size()=3 &lt; minVotersForWhistle=5</strong></p>
<pre class=" language-cpp"><code class="prism  language-cpp">missing <span class="token operator">=</span> <span class="token number">5</span> <span class="token operator">-</span> <span class="token number">3</span> <span class="token operator">=</span> <span class="token number">2</span> <span class="token comment">// 还差2台机器人</span>
numOfChannels <span class="token operator">+</span><span class="token operator">=</span> <span class="token number">2</span> <span class="token operator">*</span> <span class="token punctuation">(</span>假设每台<span class="token number">2</span>通道<span class="token punctuation">)</span> <span class="token operator">=</span> <span class="token number">4</span> <span class="token comment">// 分母增加4</span>
</code></pre>
<p>最终：</p>
<pre><code>numOfChannels = 6 + 4 = 10
12 / 10 = 1.2 &gt; 1.15 ✓ 勉强通过
</code></pre>
<p><strong>如果遇到一下情况，尽管达到了置信度可能还是没反应：</strong></p>
<ol>
<li><strong>不是所有3台都听到了口哨</strong></li>
</ol>
<ul>
<li>如果只有1-2台听到，稀释后可能不够</li>
<li>例如：只有1台听到，置信度=2，通道=2</li>
</ul>
<pre><code>totalConfidence = 2 × 2 = 4
numOfChannels = 2 + (4台没听到 × 2通道) = 10
4 / 10 = 0.4 &lt; 1.15 ✗ 不通过！
</code></pre>
<ol start="2">
<li><strong><code>useGameControllerData</code> 参数的影响</strong></li>
</ol>
<ul>
<li>如果 GC 认为场上有 5 台机器人（5v5 模式），但只有 3 台通信</li>
<li>代码会认为有 2 台"失联"，把这 2 台的通道数也加到分母上</li>
</ul>
<ol start="3">
<li><strong>时间窗口问题</strong></li>
</ol>
<ul>
<li><code>maxWhistleTimeDifference = 1000ms</code></li>
<li>如果 3 台机器人检测到口哨的时间差 &gt; 1 秒，不会被一起计算</li>
</ul>
<h3 id="根本原因总结">3.4 根本原因总结</h3>
<p><strong>分母被"人为"增大了</strong><br>
代码的设计逻辑是：</p>
<ul>
<li>如果场上应该有 5 台机器人，但只有 3 台听到口哨</li>
<li>代码会假设那 2 台"没听到"的机器人也在监听</li>
<li>把他们的通道数加到分母上，相当于"他们投了反对票"</li>
<li>这样可以防止少数机器人误判导致全队误动作</li>
</ul>
<p><strong>但这个设计在训练时（机器人数量不足5台）会导致问题</strong></p>
<h3 id="解决方案">3.5 解决方案</h3>
<p>修改<code>Config/Scenarios/Default/gameStateProvider.cfg</code>：</p>
<pre class=" language-cfg"><code class="prism  language-cfg">
# 方案1：降低最小投票人数（推荐用于训练）
minVotersForWhistle = 1; # 单台机器人就能触发
minWhistleAverageConfidence=0.5-1; #降低平均置信度阈值
  
# 方案2：根据训练人数设置
minVotersForWhistle = 3; # 3v3 训练时使用
minWhistleAverageConfidence=0.5-1; #降低平均置信度阈值
</code></pre>
<p><strong>推荐方案1</strong>，把 <code>minVotersForWhistle</code> 改成 1，这样：</p>
<ul>
<li>只要 1 台机器人听到口哨且置信度够，就能触发</li>
<li>不会有"稀释"问题</li>
<li>训练和比赛都能用（比赛时多台机器人听到只会更可靠）</li>
</ul>
<h2 id="关键代码位置">4. 关键代码位置</h2>
<h3 id="状态转换逻辑">4.1 状态转换逻辑</h3>
<p>文件：<code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code><br>
第 253-304 行：</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">// Whistle detection for waitFor* states - always active regardless of GameController status</span>
<span class="token comment">// This allows both whistle and GameController to trigger the transition to playing</span>
<span class="token keyword">switch</span><span class="token punctuation">(</span>gameState<span class="token punctuation">.</span>state<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOwnKickOff<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOpponentKickOff<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForDropBall<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOwnPenaltyKick<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOpponentPenaltyKick<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOwnPenaltyShot<span class="token operator">:</span>
	<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOpponentPenaltyShot<span class="token operator">:</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">checkForWhistle</span><span class="token punctuation">(</span>minWhistleTimestamp<span class="token punctuation">,</span> gameState<span class="token punctuation">.</span>gameControllerActive<span class="token punctuation">)</span><span class="token punctuation">)</span>
	<span class="token punctuation">{</span>
		<span class="token comment">// 切换到对应的 playing 状态</span>
		<span class="token keyword">switch</span><span class="token punctuation">(</span>gameState<span class="token punctuation">.</span>state<span class="token punctuation">)</span>
		<span class="token punctuation">{</span>
			<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOwnKickOff<span class="token operator">:</span>
				gameState<span class="token punctuation">.</span>state <span class="token operator">=</span> GameState<span class="token operator">::</span>ownKickOff<span class="token punctuation">;</span>
				<span class="token keyword">break</span><span class="token punctuation">;</span>
			<span class="token keyword">case</span> GameState<span class="token operator">::</span>waitForOpponentKickOff<span class="token operator">:</span>
				gameState<span class="token punctuation">.</span>state <span class="token operator">=</span> GameState<span class="token operator">::</span>opponentKickOff<span class="token punctuation">;</span>
				<span class="token keyword">break</span><span class="token punctuation">;</span>
<span class="token comment">// ... 其他状态</span>
		<span class="token punctuation">}</span>
		gameState<span class="token punctuation">.</span>whistled <span class="token operator">=</span> <span class="token boolean">true</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">break</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="口哨检测函数详解">4.2 口哨检测函数详解</h3>
<p>文件：<code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code>，第 558-625 行</p>
<h4 id="完整代码：">完整代码：</h4>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">bool</span> GameStateProvider<span class="token operator">::</span><span class="token function">checkForWhistle</span><span class="token punctuation">(</span><span class="token keyword">unsigned</span>  from<span class="token punctuation">,</span> <span class="token keyword">bool</span>  useGameControllerData<span class="token punctuation">)</span> <span class="token keyword">const</span>
<span class="token punctuation">{</span>
	std<span class="token operator">::</span>vector<span class="token operator">&lt;</span><span class="token keyword">const</span> Whistle<span class="token operator">*</span><span class="token operator">&gt;</span> data<span class="token punctuation">;</span> <span class="token comment">// 存储听到口哨的机器人数据</span>
	<span class="token keyword">int</span> numOfChannels <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> <span class="token comment">// 总麦克风通道数（分母）</span>
	<span class="token keyword">int</span> numOfMissingChannels <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> <span class="token comment">// 没听到口哨的机器人的通道数</span>
	size_t numOfVoters <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> <span class="token comment">// 参与投票的机器人总数</span>
<span class="token comment">// ========== 第一步：统计自己 ==========</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>theWhistle<span class="token punctuation">.</span>channelsUsedForWhistleDetection <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token operator">++</span>numOfVoters<span class="token punctuation">;</span> <span class="token comment">// 自己算一个投票者</span>
	numOfChannels <span class="token operator">+</span><span class="token operator">=</span> theWhistle<span class="token punctuation">.</span>channelsUsedForWhistleDetection<span class="token punctuation">;</span> <span class="token comment">// 加上自己的通道数</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>theWhistle<span class="token punctuation">.</span>lastTimeWhistleDetected <span class="token operator">&gt;</span> from<span class="token punctuation">)</span> <span class="token comment">// 如果自己听到了口哨</span>
		data<span class="token punctuation">.</span><span class="token function">emplace_back</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>theWhistle<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 把自己加入"听到口哨"的列表</span>
<span class="token punctuation">}</span>
<span class="token comment">// ========== 第二步：统计队友 ==========</span>
<span class="token keyword">for</span><span class="token punctuation">(</span><span class="token keyword">const</span> Teammate<span class="token operator">&amp;</span> teammate <span class="token operator">:</span> theTeamData<span class="token punctuation">.</span>teammates<span class="token punctuation">)</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>teammate<span class="token punctuation">.</span>theWhistle<span class="token punctuation">.</span>channelsUsedForWhistleDetection <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">)</span>
	<span class="token punctuation">{</span>
	<span class="token operator">++</span>numOfVoters<span class="token punctuation">;</span> <span class="token comment">// 队友算一个投票者</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>teammate<span class="token punctuation">.</span>theWhistle<span class="token punctuation">.</span>lastTimeWhistleDetected <span class="token operator">&gt;</span> from<span class="token punctuation">)</span> <span class="token comment">// 如果队友听到了口哨</span>
	<span class="token punctuation">{</span>
		numOfChannels <span class="token operator">+</span><span class="token operator">=</span> teammate<span class="token punctuation">.</span>theWhistle<span class="token punctuation">.</span>channelsUsedForWhistleDetection<span class="token punctuation">;</span>
		data<span class="token punctuation">.</span><span class="token function">emplace_back</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>teammate<span class="token punctuation">.</span>theWhistle<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 加入"听到口哨"列表</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token comment">// 队友没听到口哨</span>
	numOfMissingChannels <span class="token operator">+</span><span class="token operator">=</span> teammate<span class="token punctuation">.</span>theWhistle<span class="token punctuation">.</span>channelsUsedForWhistleDetection<span class="token punctuation">;</span> <span class="token comment">// 记录没听到的通道数</span>
	<span class="token punctuation">}</span>
<span class="token comment">// ========== 第三步：如果用GC数据，补充未通信的队友 ==========</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>useGameControllerData<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token comment">// 从GC获取己方未被罚下的球员数</span>
	size_t numOfPlayers <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
	<span class="token keyword">const</span> RoboCup<span class="token operator">::</span>TeamInfo<span class="token operator">&amp;</span> ownTeam <span class="token operator">=</span> theGameControllerData<span class="token punctuation">.</span>teams<span class="token punctuation">[</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
	<span class="token keyword">for</span><span class="token punctuation">(</span><span class="token keyword">int</span> i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> i <span class="token operator">&lt;</span> MAX_NUM_PLAYERS<span class="token punctuation">;</span> <span class="token operator">++</span>i<span class="token punctuation">)</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>ownTeam<span class="token punctuation">.</span>players<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">.</span>penalty <span class="token operator">==</span> PENALTY_NONE<span class="token punctuation">)</span>
	<span class="token operator">++</span>numOfPlayers<span class="token punctuation">;</span>
<span class="token comment">// 计算"失联"的队友数（GC说有5人，但只收到2人通信，则有2人失联）</span>
	<span class="token keyword">const</span>  size_t numOfMissingPlayers <span class="token operator">=</span> numOfPlayers <span class="token operator">-</span> std<span class="token operator">::</span><span class="token function">min</span><span class="token punctuation">(</span>numOfPlayers<span class="token punctuation">,</span> theTeamData<span class="token punctuation">.</span>teammates<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">+</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	numOfVoters <span class="token operator">+</span><span class="token operator">=</span> numOfMissingPlayers<span class="token punctuation">;</span>
	numOfMissingChannels <span class="token operator">+</span><span class="token operator">=</span> <span class="token keyword">static_cast</span><span class="token operator">&lt;</span><span class="token keyword">int</span><span class="token operator">&gt;</span><span class="token punctuation">(</span><span class="token number">2</span> <span class="token operator">*</span> numOfMissingPlayers<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 假设失联队友每人2通道</span>
<span class="token punctuation">}</span>
<span class="token comment">// ========== 第四步：关键！稀释置信度 ==========</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> minVotersForWhistle <span class="token operator">&amp;&amp;</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> numOfVoters<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token comment">// 听到口哨的人数 &lt; 5，且 &lt; 总投票人数</span>
	<span class="token comment">// 则假设那些"没听到"的人也在监听，把他们的通道数加到分母上</span>
	<span class="token keyword">const</span>  size_t missing <span class="token operator">=</span> minVotersForWhistle <span class="token operator">-</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	numOfChannels <span class="token operator">+</span><span class="token operator">=</span> <span class="token keyword">static_cast</span><span class="token operator">&lt;</span><span class="token keyword">int</span><span class="token operator">&gt;</span><span class="token punctuation">(</span>missing <span class="token operator">*</span> numOfMissingChannels <span class="token operator">/</span> <span class="token punctuation">(</span>numOfVoters <span class="token operator">-</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
<span class="token comment">// ========== 第五步：计算并判断 ==========</span>
<span class="token comment">// 按时间排序</span>
std<span class="token operator">::</span><span class="token function">sort</span><span class="token punctuation">(</span>data<span class="token punctuation">.</span><span class="token function">begin</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> data<span class="token punctuation">.</span><span class="token function">end</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword">for</span><span class="token punctuation">(</span>size_t i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> i <span class="token operator">&lt;</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token operator">++</span>i<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">float</span> totalConfidence <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">.</span>f<span class="token punctuation">;</span>
	<span class="token keyword">for</span><span class="token punctuation">(</span>size_t j <span class="token operator">=</span> i<span class="token punctuation">;</span> j <span class="token operator">&lt;</span> data<span class="token punctuation">.</span><span class="token function">size</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token operator">++</span>j<span class="token punctuation">)</span>
	<span class="token punctuation">{</span>
	<span class="token comment">// 累加置信度（置信度 × 通道数）</span>
	totalConfidence <span class="token operator">+</span><span class="token operator">=</span> data<span class="token punctuation">[</span>j<span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span>confidenceOfLastWhistleDetection
	<span class="token operator">*</span> <span class="token keyword">static_cast</span><span class="token operator">&lt;</span><span class="token keyword">float</span><span class="token operator">&gt;</span><span class="token punctuation">(</span>data<span class="token punctuation">[</span>j<span class="token punctuation">]</span><span class="token operator">-</span><span class="token operator">&gt;</span>channelsUsedForWhistleDetection<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token comment">// 最终判断：totalConfidence / numOfChannels &gt; 1.15 ?</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>totalConfidence <span class="token operator">/</span> numOfChannels <span class="token operator">&gt;</span> minWhistleAverageConfidence<span class="token punctuation">)</span>
		<span class="token keyword">return</span> <span class="token boolean">true</span><span class="token punctuation">;</span> <span class="token comment">// 检测到口哨！</span>
	<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
<span class="token keyword">return</span> <span class="token boolean">false</span><span class="token punctuation">;</span> <span class="token comment">// 没检测到</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="逐步解释：">逐步解释：</h4>
<p><strong>变量含义：</strong></p>

<table>
<thead>
<tr>
<th>变量</th>
<th>含义</th>
<th>例子</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>data</code></td>
<td>听到口哨的机器人列表</td>
<td>3台机器人中有2台听到 → data.size()=2</td>
</tr>
<tr>
<td><code>numOfChannels</code></td>
<td>总麦克风通道数（计算时的分母）</td>
<td>每台机器人2通道，3台=6通道</td>
</tr>
<tr>
<td><code>numOfMissingChannels</code></td>
<td>没听到口哨的机器人的通道数</td>
<td>1台没听到，2通道</td>
</tr>
<tr>
<td><code>numOfVoters</code></td>
<td>参与投票的机器人总数</td>
<td>3台机器人 → numOfVoters=3</td>
</tr>
</tbody>
</table><p><strong>最终判断公式：</strong></p>
<pre><code>totalConfidence / numOfChannels &gt; minWhistleAverageConfidence (1.15)
</code></pre>
<p>即：<code>(置信度 × 通道数的总和) / 总通道数 &gt; 1.15</code></p>
<h2 id="simrobot-中测试口哨检测">5. SimRobot 中测试口哨检测</h2>
<h3 id="测试步骤">5.1 测试步骤</h3>
<ol>
<li>启动 SimRobot(路径/Build/Linux/SimRobot/Develop)</li>
<li>选择一个.ros3文件</li>
<li>我目前用的是（/Config/Scenes/OtherScenes/OneTeamReferee.con）(用来训练手势识别的。也可以用前一个目录的.ros3文档)</li>
<li>随即选一个robot1/data/audio/representation/Whistle</li>
<li>执行 <code>ar true</code>（启用自主裁判模式）</li>
<li>执行 <code>GameState off</code>（关闭 GC 数据接收）</li>
<li>因为在simrobot当中，gc playing发布之后，gc端和口哨是同时发布的。既然要测试口哨，所以要执行5/6指令关闭gc端</li>
<li>确保机器人处于 Set 状态（waitForOwnKickOff 等）</li>
<li>在终端输入<code>gc playing</code></li>
<li>观察 <code>Whistle</code> 和 <code>GameState</code> 的变化</li>
</ol>
<h3 id="需要查看的-representation">5.2 需要查看的 Representation</h3>
<pre><code>在 SimRobot 左侧边栏展开机器人节点，查看：
</code></pre>

<table>
<thead>
<tr>
<th>Representation</th>
<th>关键字段</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>Whistle</code></td>
<td><code>confidenceOfLastWhistleDetection</code></td>
<td>置信度，需要 &gt; 1.15 ,一般显示为2.0</td>
</tr>
<tr>
<td><code>Whistle</code></td>
<td><code>channelsUsedForWhistleDetection</code></td>
<td>麦克风通道数，应该 &gt; 0 ,一般变为2/4</td>
</tr>
<tr>
<td><code>Whistle</code></td>
<td><code>lastTimeWhistleDetected</code></td>
<td>最后检测到口哨的时间戳</td>
</tr>
<tr>
<td><code>GameState</code></td>
<td><code>state</code></td>
<td>当前状态（waitForOwnKickOff → ownKickOff）</td>
</tr>
<tr>
<td><code>GameState</code></td>
<td><code>whistled</code></td>
<td>是否通过口哨触发了状态转换</td>
</tr>
</tbody>
</table>
    </div>
  </div>
</body>

</html>
