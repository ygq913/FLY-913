<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GC3-analyse</title>
  <link rel="stylesheet" href="https://stackedit.io/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__left">
    <div class="stackedit__toc">
      
<ul>
<li>
<ul>
<li><a href="#gamecontroller3-完整指南">GameController3 完整指南</a></li>
<li><a href="#、获取并编辑gamecontroller3：">1、获取并编辑GameController3：</a></li>
<li><a href="#、2025规则更新要点部分更新的规则">2、2025规则更新要点(部分更新的规则)</a></li>
<li><a href="#gc模式选择说明">3.GC模式选择说明</a></li>
<li><a href="#比赛流程详解（2025年比赛）">4. 比赛流程详解（2025年比赛）</a></li>
<li><a href="#关键代码解析">5. 关键代码解析</a></li>
<li><a href="#三种出界状态详解">6. 三种出界状态详解</a></li>
<li><a href="#附录：代码流程图">附录：代码流程图</a></li>
<li><a href="#参考资料">参考资料</a></li>
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <h2 id="gamecontroller3-完整指南">GameController3 完整指南</h2>
<p><strong>目录</strong><br>
1.获取并编辑GameController3<br>
2.2025规则更新要点<br>
3.GC模式选择说明<br>
4.比赛流程详解<br>
5.基于gc更新的关键代码解析<br>
6.三种出界状态详解</p>
<h2 id="、获取并编辑gamecontroller3："><strong>1、获取并编辑GameController3：</strong></h2>
<p><strong>1.1获取源码：</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#克隆官方仓库</span>
<span class="token function">git</span> clone https://github.com/RoboCup-		SPL/GameController3.git
<span class="token function">cd</span>  GameController3
</code></pre>
<p><strong>1.2环境要求</strong><br>
根据 <code>GameController3/README.md</code>，需要安装以下依赖：</p>
<ul>
<li><strong>Rust</strong> (建议1.77.2或更新版本)</li>
<li><strong>Node.js 和 npm</strong></li>
<li><strong>libclang</strong> (用于bindgen)</li>
<li><strong>Tauri依赖</strong> (参考 <a href="https://tauri.app/start/prerequisites/">https://tauri.app/start/prerequisites/</a>)</li>
</ul>
<p><strong>1.3安装/升级Rust</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 安装rustup（如果没有）</span>
curl  --proto  <span class="token string">'=https'</span>  --tlsv1.2  -sSf  https://sh.rustup.rs <span class="token operator">|</span> sh  -s  --  -y
<span class="token comment"># 加载环境变量</span>
<span class="token function">source</span>  <span class="token string">"<span class="token variable">$HOME</span>/.cargo/env"</span>
<span class="token comment"># 验证版本</span>
rustc  --version  <span class="token comment"># 应该 &gt;= 1.77.2</span>
</code></pre>
<p><strong>1.4编译步骤</strong><br>
<strong>步骤1：安装tauri-cli(开发模式需要)</strong></p>
<pre class=" language-bash"><code class="prism  language-bash">cargo <span class="token function">install</span> tauri-cli
</code></pre>
<p><strong>步骤2：编译前端</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span> frontend
<span class="token function">npm</span> ci					<span class="token comment">#安装依赖</span>
<span class="token function">npm</span> run build		<span class="token comment">#编译前端</span>
<span class="token function">cd</span> <span class="token punctuation">..</span>
</code></pre>
<p><strong>步骤3：编译后端</strong></p>
<pre class=" language-bash"><code class="prism  language-bash">cargo build -r    		<span class="token comment">#-r		表示release模式</span>
</code></pre>
<p><strong>1.5运行GameController</strong></p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment">#方式1：通过cargo运行：</span>
cargo run -r
<span class="token comment">#方式2：直接运行编译好的二进制文件或进入该目录</span>
./tartet/release/gamer_controller_app
</code></pre>
<h2 id="、2025规则更新要点部分更新的规则"><strong>2、2025规则更新要点(部分更新的规则)</strong></h2>
<p>根据2025-6月发布的规则文件，主要变更如下（**<strong><strong>该部分只适用2025年更新规则</strong>）：</strong></p>
<h3 id="ready信号延迟调整">2.1 Ready信号延迟调整</h3>
<ul>
<li>Champions Cup：延迟增加到 45秒</li>
<li>Challenge Shield：延迟增加到 35秒</li>
<li>Ready视觉信号现在也适用于Challenge Shield</li>
</ul>
<h3 id="free-kick踢球队伍通知方式（最重要变更">2.2  Free Kick踢球队伍通知方式（最重要变更)</h3>
<ul>
<li><strong>Champions Cup模式</strong>：free kick的踢球队伍通过<strong>裁判视觉手势</strong>通知</li>
<li><strong>GameController不再向机器人发送踢球队伍信息</strong></li>
<li>机器人必须识别裁判手势来判断哪队踢球</li>
</ul>
<h3 id="standby状态下的motion处罚">2.3 Standby状态下的Motion处罚</h3>
<ul>
<li>错误检测Ready信号导致的Motion处罚变成<strong>全队处罚</strong></li>
<li>处罚时间缩短</li>
</ul>
<h3 id="团队通信消息预算">2.4 团队通信消息预算</h3>
<ul>
<li>Standby状态下发送的通信数据包会<strong>消耗</strong>消息预算</li>
</ul>
<h3 id="未执行free-kick的后果">2.5 未执行Free Kick的后果</h3>
<ul>
<li>如果一支队伍未能执行free kick，对方可以<strong>直接尝试进球</strong></li>
</ul>
<h3 id="间接踢球规则扩展">2.6 间接踢球规则扩展</h3>
<ul>
<li>
<p>2024年Champions Cup的间接踢球规则现在也适用于Challenge Shield</p>
</li>
<li>
<p>Challenge Shield队伍可使用fallback选项</p>
</li>
</ul>
<h3 id="处罚可提前取消">2.7 处罚可提前取消</h3>
<ul>
<li>裁判错误给出的处罚可以提前取消</li>
</ul>
<h3 id="球停止规则">2.8 球停止规则</h3>
<ul>
<li>每半场可稍微延长，让进攻局面完成</li>
</ul>
<h2 id="gc模式选择说明"><strong>3.GC模式选择说明</strong></h2>
<h3 id="四种模式对比">3.1 四种模式对比</h3>
<p>| 模式 | 机器人数量 | hideKickingSide | 需要识别裁判手势？ | 适用场景 |</p>
<p>|------|----------------|--------------------|-------------------------|-------------|</p>
<p>| Champions Cup | 7人 | true | ✅ 需要 | 正式比赛(7v7) |</p>
<p>| Champions Cup 5 vs. 5 | 5人 | true | ✅ 需要 | 正式比赛(5v5) |</p>
<p>| Challenge Shield | 5人 | false | ❌ 不需要 | 新队伍/入门级 |</p>
<p>| Most Passes Leaderboard | 2人 | false | ❌ 不需要 | 技术挑战赛 |</p>
<h3 id="配置文件位置">3.2 配置文件位置</h3>
<ul>
<li>Champions Cup: <code>GameController3/config/champions_cup/params.yaml</code></li>
<li>Champions Cup 5v5: <code>GameController3/config/champions_cup_5/params.yaml</code></li>
<li>Challenge Shield: <code>GameController3/config/challenge_shield/params.yaml</code></li>
</ul>
<h3 id="关键配置项">3.3 关键配置项</h3>
<pre><code># Champions Cup / Champions Cup 5v5（我们是这组）
hideKickingSide: true  # 隐藏踢球队伍信息
# Challenge Shield
hideKickingSide: false  # 不隐藏踢球队伍信息
</code></pre>
<hr>
<h2 id="比赛流程详解（2025年比赛）">4. 比赛流程详解（2025年比赛）</h2>
<h3 id="出界处理流程（以kick-in为例）">4.1 出界处理流程（以Kick-in为例）</h3>
<h4 id="gc操作员视角">GC操作员视角</h4>
<ol>
<li>裁判看到球出界，喊 “Kick-in blue”（蓝队踢）</li>
<li>GC操作员听到后，点击<strong>蓝队下方</strong>的 “Kick-in” 按钮</li>
<li>GC内部记录：<code>kicking_side = Some(蓝队)</code></li>
<li>GC发送数据包给机器人</li>
</ol>
<h4 id="机器人视角（champions-cup模式）">机器人视角（Champions Cup模式）</h4>
<ol>
<li>收到数据包：</li>
</ol>
<ul>
<li><code>setPlay = 4</code> (SET_PLAY_KICK_IN) → 知道是Kick-in</li>
<li><code>kickingTeam = 255</code> (KICKING_TEAM_NONE) → <strong>不知道哪队踢球</strong></li>
</ul>
<ol start="2">
<li>机器人需要：</li>
</ol>
<ul>
<li>找到裁判</li>
<li>识别裁判举起的手臂</li>
<li>判断手臂指向哪个球门</li>
<li>从而确定哪队踢球</li>
</ul>
<h4 id="机器人视角（challenge-shield模式）（简要说明，我们不参加）">机器人视角（Challenge Shield模式）（简要说明，我们不参加）</h4>
<ol>
<li>收到数据包：</li>
</ol>
<ul>
<li><code>setPlay = 4</code> (SET_PLAY_KICK_IN) → 知道是Kick-in</li>
<li><code>kickingTeam = 5</code> (队伍编号) → <strong>直接知道哪队踢球</strong></li>
</ul>
<ol start="2">
<li>不需要识别裁判手势</li>
</ol>
<h3 id="裁判手势说明">4.2 裁判手势说明</h3>
<p>根据规则3.9.1节：</p>
<blockquote>
<p>“The gesture consists of raising one arm horizontally and sideways, parallel to the field line… The raised arm is the one pointing towards the defending team’s goal”</p>
</blockquote>
<ul>
<li>裁判举起一只手臂，水平向侧面伸出，与场地线平行</li>
<li>举起的手臂指向<strong>防守方的球门</strong>（即不踢球的那队的球门）</li>
<li>手势保持至少5秒</li>
</ul>
<hr>
<h2 id="关键代码解析">5. 关键代码解析</h2>
<h3 id="核心文件位置">5.1 核心文件位置</h3>
<pre><code>GameController3/
├── game_controller_msgs/
│ ├── src/
│ │ └── control_message.rs #  发送给机器人的数据包构建
│ └── headers/
│ └── RoboCupGameControlData.h #  数据包结构定义和常量
├── game_controller_core/
│ └── src/
│ └── actions/
│ └── start_set_play.rs # Set Play动作的执行和合法性检查
└── config/
├── champions_cup/params.yaml # Champions Cup配置
├── champions_cup_5/params.yaml # Champions Cup 5v5配置
└── challenge_shield/params.yaml # Challenge Shield配置
</code></pre>
<h3 id="常量定义">5.2 常量定义</h3>
<p><strong>文件</strong>: <code>GameController3/game_controller_msgs/headers/RoboCupGameControlData.h</code></p>
<pre class=" language-c"><code class="prism  language-c"><span class="token comment">// Set Play 类型常量（第45-50行）</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_NONE  0 </span><span class="token comment">// 无 set play</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_GOAL_KICK  1 </span><span class="token comment">// Goal Kick（球门球）</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_PUSHING_FREE_KICK  2 </span><span class="token comment">// Pushing Free Kick</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_CORNER_KICK  3 </span><span class="token comment">// Corner Kick（角球）</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_KICK_IN  4 </span><span class="token comment">// Kick-in（边线球）</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  SET_PLAY_PENALTY_KICK  5 </span><span class="token comment">// Penalty Kick（点球）</span>
<span class="token comment">// 踢球队伍常量（第51行）</span>
<span class="token macro property">#<span class="token directive keyword">define</span>  KICKING_TEAM_NONE  255 </span><span class="token comment">// 未知/隐藏</span>
</code></pre>
<p><strong>数据包结构</strong>（第98-104行）:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> RoboCupGameControlData <span class="token punctuation">{</span>
<span class="token comment">// ... 其他字段 ...</span>
uint8_t setPlay<span class="token punctuation">;</span> <span class="token comment">// 当前 set play 类型 (0-5)</span>
uint8_t firstHalf<span class="token punctuation">;</span> <span class="token comment">// 是否上半场</span>
uint8_t kickingTeam<span class="token punctuation">;</span> <span class="token comment">// 踢球队伍编号，或255表示未知</span>
int16_t secsRemaining<span class="token punctuation">;</span> <span class="token comment">// 剩余秒数</span>
int16_t secondaryTime<span class="token punctuation">;</span> <span class="token comment">// 次要计时器（如free kick倒计时）</span>
<span class="token comment">// ... 其他字段 ...</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<h3 id="hide_kicking_side-判断逻辑">5.3 hide_kicking_side 判断逻辑</h3>
<p><strong>文件</strong>: <code>GameController3/game_controller_msgs/src/control_message.rs</code></p>
<p><strong>位置</strong>: 第163-170行</p>
<pre class=" language-rust"><code class="prism  language-rust"><span class="token keyword">let</span>  hide_kicking_side <span class="token operator">=</span> <span class="token operator">!</span>to_monitor  <span class="token comment">// 条件1: 不是发给监控程序</span>
<span class="token operator">&amp;&amp;</span> params<span class="token punctuation">.</span>competition<span class="token punctuation">.</span>hide_kicking_side <span class="token comment">// 条件2: 配置文件中 hideKickingSide: true</span>
<span class="token operator">&amp;&amp;</span> game<span class="token punctuation">.</span>phase <span class="token operator">!=</span> Phase<span class="token punctuation">:</span><span class="token punctuation">:</span>PenaltyShootout <span class="token comment">// 条件3: 不是点球大战</span>
<span class="token operator">&amp;&amp;</span> <span class="token punctuation">(</span><span class="token punctuation">(</span>game<span class="token punctuation">.</span>state <span class="token operator">==</span> State<span class="token punctuation">:</span><span class="token punctuation">:</span>Ready
<span class="token operator">||</span> game<span class="token punctuation">.</span>state <span class="token operator">==</span> State<span class="token punctuation">:</span><span class="token punctuation">:</span>Set
<span class="token operator">||</span> game<span class="token punctuation">.</span>state <span class="token operator">==</span> State<span class="token punctuation">:</span><span class="token punctuation">:</span>Playing<span class="token punctuation">)</span> <span class="token comment">// 条件4: 在 Ready/Set/Playing 状态</span>
<span class="token operator">&amp;&amp;</span> game<span class="token punctuation">.</span>set_play <span class="token operator">!=</span> SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>KickOff<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 条件5: 不是开球</span>
</code></pre>
<p><strong>解释</strong>：当以上5个条件<strong>全部满足</strong>时，<code>hide_kicking_side = true</code>，机器人将收不到踢球队伍信息。</p>
<h3 id="set_play-字段设置与5.2节结合">5.4 set_play 字段设置(与5.2节结合)</h3>
<p><strong>文件</strong>: <code>GameController3/game_controller_msgs/src/control_message.rs</code></p>
<p><strong>位置</strong>: 第193-200行</p>
<pre class=" language-rust"><code class="prism  language-rust">set_play<span class="token punctuation">:</span> <span class="token keyword">match</span>  game<span class="token punctuation">.</span>set_play <span class="token punctuation">{</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>NoSetPlay <span class="token operator">|</span> SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>KickOff <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_NONE<span class="token punctuation">,</span> <span class="token comment">// 无/开球 → 0</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>KickIn <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_KICK_IN<span class="token punctuation">,</span> <span class="token comment">// Kick-in → 4</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>GoalKick <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_GOAL_KICK<span class="token punctuation">,</span> <span class="token comment">// Goal Kick → 1</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>CornerKick <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_CORNER_KICK<span class="token punctuation">,</span> <span class="token comment">// Corner Kick → 3</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>PushingFreeKick <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_PUSHING_FREE_KICK<span class="token punctuation">,</span> <span class="token comment">// Pushing Free Kick → 2</span>
SetPlay<span class="token punctuation">:</span><span class="token punctuation">:</span>PenaltyKick <span class="token operator">=</span><span class="token operator">&gt;</span> SET_PLAY_PENALTY_KICK<span class="token punctuation">,</span> <span class="token comment">// Penalty Kick → 5</span>
<span class="token punctuation">}</span><span class="token punctuation">,</span>
</code></pre>
<p><strong>解释</strong>：这里将内部枚举转换为发送给机器人的数字常量。机器人<strong>总是能收到</strong>这个信息。</p>
<h3 id="kicking_team-字段设置（⭐最关键）">5.5 kicking_team 字段设置（⭐最关键）</h3>
<p><strong>文件</strong>: <code>GameController3/game_controller_msgs/src/control_message.rs</code></p>
<p><strong>位置</strong>: 第202-204行</p>
<pre class=" language-rust"><code class="prism  language-rust">
kicking_team<span class="token punctuation">:</span> game
<span class="token punctuation">.</span>kicking_side <span class="token comment">// 获取GC内部记录的踢球队伍</span>
<span class="token punctuation">.</span><span class="token function">filter</span><span class="token punctuation">(</span><span class="token operator">|</span>_<span class="token operator">|</span> <span class="token operator">!</span>hide_kicking_side<span class="token punctuation">)</span> <span class="token comment">// 如果hide_kicking_side=true，则过滤掉</span>
<span class="token punctuation">.</span><span class="token function">map_or</span><span class="token punctuation">(</span>KICKING_TEAM_NONE<span class="token punctuation">,</span> <span class="token operator">|</span>side<span class="token operator">|</span> params<span class="token punctuation">.</span>game<span class="token punctuation">.</span>teams<span class="token punctuation">[</span>side<span class="token punctuation">]</span><span class="token punctuation">.</span>number<span class="token punctuation">)</span><span class="token punctuation">,</span>
</code></pre>
<p><strong>解释</strong>：</p>
<ol>
<li><code>game.kicking_side</code> - GC内部记录的踢球队伍（Some(Home)或Some(Away)）</li>
<li><code>.filter(|_| !hide_kicking_side)</code> - 如果<code>hide_kicking_side = true</code>，返回<code>None</code></li>
<li><code>.map_or(KICKING_TEAM_NONE, ...)</code> - 如果是<code>None</code>，返回255；否则返回队伍编号</li>
</ol>
<hr>
<h2 id="三种出界状态详解">6. 三种出界状态详解</h2>
<h3 id="kick-in（边线出界）">6.1 Kick-in（边线出界）</h3>
<p><strong>触发条件</strong>：球越过边线（touchline）（裁判来判断并给出口令）</p>
<p><strong>球的位置</strong>：放在球出界的边线位置</p>
<p><strong>GC操作员操作</strong>：点击<strong>获得踢球权的队伍</strong>下方的 “Kick-in” 按钮</p>
<p><strong>机器人收到的数据包</strong>：</p>

<table>
<thead>
<tr>
<th>字段</th>
<th>Champions Cup</th>
<th>Challenge Shield</th>
</tr>
</thead>
<tbody>
<tr>
<td>setPlay</td>
<td>4 (SET_PLAY_KICK_IN)</td>
<td>4 (SET_PLAY_KICK_IN)</td>
</tr>
<tr>
<td>kickingTeam</td>
<td><strong>255 (未知)</strong></td>
<td>队伍编号</td>
</tr>
<tr>
<td>secondaryTime</td>
<td>30秒倒计时</td>
<td>30秒倒计时</td>
</tr>
</tbody>
</table><h3 id="goal-kick（球门球）">6.2 Goal Kick（球门球）</h3>
<p><strong>触发条件</strong>：球越过球门线，且最后触球的是<strong>进攻方</strong><br>
<strong>球的位置</strong>：放在球门区角落（goal area corner）</p>
<p><strong>GC操作员操作</strong>：点击**防守方（获得踢球权）**下方的 “Goal Kick” 按钮<br>
<strong>机器人收到的数据包</strong>：</p>

<table>
<thead>
<tr>
<th>字段</th>
<th>Champions Cup</th>
<th>Challenge Shield</th>
</tr>
</thead>
<tbody>
<tr>
<td>setPlay</td>
<td>1 (SET_PLAY_GOAL_KICK)</td>
<td>1 (SET_PLAY_GOAL_KICK)</td>
</tr>
<tr>
<td>kickingTeam</td>
<td><strong>255 (未知)</strong></td>
<td>队伍编号</td>
</tr>
<tr>
<td>secondaryTime</td>
<td>30秒倒计时</td>
<td>30秒倒计时</td>
</tr>
</tbody>
</table><h3 id="corner-kick（角球）">6.3 Corner Kick（角球）</h3>
<p><strong>触发条件</strong>：球越过球门线，且最后触球的是<strong>防守方</strong></p>
<p><strong>球的位置</strong>：放在角球点（corner）</p>
<p><strong>GC操作员操作</strong>：点击**进攻方（获得踢球权）**下方的 “Corner Kick” 按钮<br>
<strong>机器人收到的数据包</strong>：</p>

<table>
<thead>
<tr>
<th>字段</th>
<th>Champions Cup</th>
<th>Challenge Shield</th>
</tr>
</thead>
<tbody>
<tr>
<td>setPlay</td>
<td>3 (SET_PLAY_CORNER_KICK)</td>
<td>3 (SET_PLAY_CORNER_KICK)</td>
</tr>
<tr>
<td>kickingTeam</td>
<td><strong>255 (未知)</strong></td>
<td>队伍编号</td>
</tr>
<tr>
<td>secondaryTime</td>
<td>30秒倒计时</td>
<td>30秒倒计时</td>
</tr>
</tbody>
</table><h2 id="附录：代码流程图">附录：代码流程图</h2>
<pre><code>
	GC操作员点击 Goal Kick/Kick-in/Corner Kick 按钮
						↓
		game.set_play = SetPlay::GoalKick/KickIn/CornerKick
		game.kicking_side = Some(Home) 或 Some(Away)
						↓
				构建 ControlMessage
						↓
			检查 hide_kicking_side 条件
						↓
		┌───────────────┴───────────────┐
		↓ 								↓
Champions Cup           		Challenge Shield
hide_kicking_side=true    	hide_kicking_side=false
		↓ 								↓
kickingTeam = 255 				kickingTeam = 队伍编号
		↓  								↓
机器人不知道哪队踢球 				机器人直接知道哪队踢球
		↓ 								↓
需要识别裁判手势 无需额外操作
</code></pre>
<hr>
<h2 id="参考资料">参考资料</h2>
<ul>
<li>官方GameController仓库: <a href="https://github.com/RoboCup-SPL/GameController3">https://github.com/RoboCup-SPL/GameController3</a></li>
<li>B-Human 2025 Changelog: <a href="https://docs.b-human.de/master/changelog/2025/">https://docs.b-human.de/master/changelog/2025/</a></li>
<li>RoboCup SPL 2025规则文档</li>
</ul>

    </div>
  </div>
</body>

</html>
