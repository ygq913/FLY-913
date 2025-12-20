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
<li><a href="#比赛流程详解（2026年比赛）">4. 比赛流程详解（2026年比赛）</a></li>
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
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 克隆官方仓库</span>
<span class="token function">git</span>  clone  https://github.com/RoboCup-SPL/GameController3.git
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
<h3 id="安装依赖（ubuntudebian-linux）">1.3 安装依赖（Ubuntu/Debian Linux）</h3>
<h4 id="安装-node.js-和-npm">1.3.1 安装 Node.js 和 npm</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 方式1：使用apt安装</span>
<span class="token function">sudo</span> apt update
<span class="token function">sudo</span> apt <span class="token function">install</span> nodejs <span class="token function">npm</span>
<span class="token comment"># 方式2：使用nvm安装（推荐，可以管理多版本）</span>
curl  -o-  https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh <span class="token operator">|</span> <span class="token function">bash</span>
<span class="token function">source</span>  ~/.bashrc
nvm  <span class="token function">install</span>  --lts
nvm  use  --lts
<span class="token comment"># 验证安装</span>
node  --version
<span class="token function">npm</span>  --version
</code></pre>
<h4 id="安装-libclang">1.3.2 安装 libclang</h4>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span>  apt  <span class="token function">install</span>  libclang-dev
</code></pre>
<h4 id="安装-tauri-依赖">1.3.3 安装 Tauri 依赖</h4>
<p>根据 Tauri 官方文档 (<a href="https://tauri.app/start/prerequisites/">https://tauri.app/start/prerequisites/</a>)，Ubuntu/Debian 需要安装：</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">sudo</span>  apt  update
<span class="token function">sudo</span>  apt  <span class="token function">install</span>  libwebkit2gtk-4.1-dev \
build-essential \
curl \
<span class="token function">wget</span> \
<span class="token function">file</span> \
libxdo-dev \
libssl-dev \
libayatana-appindicator3-dev \
librsvg2-dev
</code></pre>
<h4 id="安装升级-rust">1.3.4 安装/升级 Rust</h4>
<p>如果系统 Rust 版本过低或未安装，使用 rustup：</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 安装rustup（如果没有）</span>
curl  --proto  <span class="token string">'=https'</span>  --tlsv1.2  -sSf  https://sh.rustup.rs <span class="token operator">|</span> sh  -s  --  -y
  
<span class="token comment"># 加载环境变量</span>
<span class="token function">source</span>  <span class="token string">"<span class="token variable">$HOME</span>/.cargo/env"</span>
  
<span class="token comment"># 验证版本</span>
rustc  --version  <span class="token comment"># 应该 &gt;= 1.77.2</span>
  
<span class="token comment"># 如果版本过低，升级Rust</span>
rustup  update  stable
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
<li>Champions Cup 5v5(我们参加这组): <code>GameController3/config/champions_cup_5/params.yaml</code></li>
<li>Challenge Shield: <code>GameController3/config/challenge_shield/params.yaml</code></li>
</ul>
<h3 id="关键配置项">3.3 关键配置项</h3>
<pre><code># Champions Cup / Champions Cup 5v5（我们是这组）
hideKickingSide: true  # 隐藏踢球队伍信息
# Challenge Shield
hideKickingSide: false  # 不隐藏踢球队伍信息
</code></pre>
<hr>
<h2 id="比赛流程详解（2026年比赛）">4. 比赛流程详解（2026年比赛）</h2>
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
<h3 id="kick-in（边线出界）——-需要识别裁判手势">6.1 Kick-in（边线出界）—— 需要识别裁判手势</h3>
<h4 id="触发条件">6.1.1 触发条件</h4>
<p>球越过边线（touchline），无论是哪支队伍踢出去的。</p>
<h4 id="球的位置">6.1.2 球的位置</h4>
<p>裁判将球放在球出界的边线位置。</p>
<h4 id="gc操作员操作流程">6.1.3 GC操作员操作流程</h4>
<ol>
<li>裁判观察是哪支队伍最后触球，喊出 “Kick-in &lt;颜色&gt;”（如 “Kick-in blue”）</li>
<li>GC操作员听到后，点击<strong>获得踢球权的队伍</strong>下方的 “Kick-in” 按钮</li>
<li>GC内部记录 <code>game.kicking_side = Some(该队伍)</code></li>
<li>GC构建数据包并发送给机器人</li>
</ol>
<h4 id="机器人收到的数据包">6.1.4 机器人收到的数据包</h4>

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
</table><h4 id="⭐-机器人判断踢球队伍的方式（champions-cup模式）">6.1.5 ⭐ 机器人判断踢球队伍的方式（Champions Cup模式）</h4>
<p>由于 <code>kickingTeam = 255</code>，机器人<strong>必须通过识别裁判手势</strong>来判断哪队踢球：<br>
<strong>B-Human 机器人端代码</strong>：</p>
<p><strong>文件位置</strong>: <code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code></p>
<p><strong>第861-879行 - 通过裁判手势判断Kick-in踢球队伍</strong>：</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">void</span> GameStateProvider<span class="token operator">::</span><span class="token function">updateKickingTeamByRefereeSignal</span><span class="token punctuation">(</span>GameState<span class="token operator">&amp;</span>  gameState<span class="token punctuation">)</span>

<span class="token punctuation">{</span>
<span class="token comment">// Only execute during free kick when the kicking team is hidden.</span>
<span class="token comment">// 只在free kick且踢球队伍被隐藏时执行</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>gameState<span class="token punctuation">.</span>kickingTeamKnown <span class="token operator">||</span> <span class="token operator">!</span>gameState<span class="token punctuation">.</span><span class="token function">isKickIn</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
	<span class="token keyword">return</span><span class="token punctuation">;</span>

<span class="token comment">// 检测裁判手势：leftHandTeam表示左边队伍，根据手势方向判断</span>
<span class="token comment">// kickInRight = 裁判手臂指向右边球门 → 左边队伍踢球</span>
<span class="token comment">// kickInLeft = 裁判手臂指向左边球门 → 右边队伍踢球</span>
<span class="token keyword">const</span>  <span class="token keyword">bool</span> forOwnTeam <span class="token operator">=</span> <span class="token function">checkForRefereeSignal</span><span class="token punctuation">(</span>gameState<span class="token punctuation">,</span>
gameState<span class="token punctuation">.</span>leftHandTeam <span class="token operator">?</span> RefereeSignal<span class="token operator">::</span>kickInRight <span class="token operator">:</span> RefereeSignal<span class="token operator">::</span>kickInLeft<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">const</span>  <span class="token keyword">bool</span> forOpponentTeam <span class="token operator">=</span> <span class="token function">checkForRefereeSignal</span><span class="token punctuation">(</span>gameState<span class="token punctuation">,</span>
gameState<span class="token punctuation">.</span>leftHandTeam <span class="token operator">?</span> RefereeSignal<span class="token operator">::</span>kickInLeft <span class="token operator">:</span> RefereeSignal<span class="token operator">::</span>kickInRight<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>forOwnTeam<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
isKickingTeam <span class="token operator">=</span> <span class="token boolean">true</span><span class="token punctuation">;</span> <span class="token comment">// 我方踢球</span>
gameState<span class="token punctuation">.</span>state <span class="token operator">=</span> GameState<span class="token operator">::</span>ownKickIn<span class="token punctuation">;</span> <span class="token comment">// 更新状态为我方Kick-in</span>
<span class="token punctuation">}</span>
<span class="token keyword">if</span><span class="token punctuation">(</span>forOpponentTeam<span class="token punctuation">)</span> <span class="token comment">// or for both teams</span>
<span class="token punctuation">{</span>
isKickingTeam <span class="token operator">=</span> <span class="token boolean">false</span><span class="token punctuation">;</span> <span class="token comment">// 对方踢球</span>
gameState<span class="token punctuation">.</span>state <span class="token operator">=</span> GameState<span class="token operator">::</span>opponentKickIn<span class="token punctuation">;</span> <span class="token comment">// 更新状态为对方Kick-in</span>
<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>第991-997行 - 处理GC数据包中的Kick-in</strong>：</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">else</span>  <span class="token keyword">if</span><span class="token punctuation">(</span>gameControllerData<span class="token punctuation">.</span>setPlay <span class="token operator">==</span> SET_PLAY_KICK_IN<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
<span class="token comment">// 如果kickingTeam无效(=255)且不是之前的Kick-in状态，默认假设对方踢球</span>
<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>isKickingTeamValid <span class="token operator">&amp;&amp;</span> <span class="token operator">!</span>GameState<span class="token operator">::</span><span class="token function">isKickIn</span><span class="token punctuation">(</span>lastGameControllerState<span class="token punctuation">)</span><span class="token punctuation">)</span>
isKickingTeam <span class="token operator">=</span> <span class="token boolean">false</span><span class="token punctuation">;</span> <span class="token comment">// 默认假设对方踢球，等待裁判手势确认</span>
<span class="token keyword">return</span> isKickingTeam <span class="token operator">?</span> GameState<span class="token operator">::</span>ownKickIn <span class="token operator">:</span> GameState<span class="token operator">::</span>opponentKickIn<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>代码解释</strong>：</p>
<ol>
<li>机器人收到 <code>setPlay = SET_PLAY_KICK_IN</code> 且 <code>kickingTeam = 255</code></li>
<li>默认先假设对方踢球（<code>isKickingTeam = false</code>）</li>
<li>调用 <code>updateKickingTeamByRefereeSignal()</code> 检测裁判手势</li>
<li>根据检测到的手势方向更新 <code>isKickingTeam</code> 和游戏状态</li>
</ol>
<p><strong>判断流程图</strong>：</p>
<pre><code>			机器人收到 setPlay=4, kickingTeam=255
							↓
			默认假设对方踢球 (isKickingTeam=false)
							↓
			调用 updateKickingTeamByRefereeSignal()
							↓
				检测裁判手势 (RefereeSignal)
							↓
				  ┌─────────┴─────────┐
				  ↓ 				  ↓
			检测到kickInRight 	检测到kickInLeft
			(手臂指向右球门) 		(手臂指向左球门)
				  ↓                   ↓
			左边队伍踢球 			右边队伍踢球
				  ↓		 			  ↓
          更新 isKickingTeam 和 gameState.state
</code></pre>
<hr>
<h3 id="goal-kick（球门球）——-根据球位置判断">6.2 Goal Kick（球门球）—— 根据球位置判断</h3>
<h4 id="触发条件-1">6.2.1 触发条件</h4>
<p>球越过球门线（goal line），且最后触球的是<strong>进攻方</strong>（即球被踢向对方球门但出界）。</p>
<h4 id="球的位置-1">6.2.2 球的位置</h4>
<p>裁判将球放在<strong>球门区角落</strong>（goal area corner），即球门区线与球门线的交点内侧。</p>
<h4 id="gc操作员操作流程-1">6.2.3 GC操作员操作流程</h4>
<ol>
<li>裁判观察到进攻方将球踢出球门线，喊出 “Goal Kick &lt;颜色&gt;”（例如 “Goal Kick red”）</li>
<li>GC操作员点击**防守方（获得踢球权）**下方的 “Goal Kick” 按钮</li>
<li>裁判将球放在球门区角落</li>
</ol>
<h4 id="机器人收到的数据包-1">6.2.4 机器人收到的数据包</h4>

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
</table><h4 id="⭐-机器人判断踢球队伍的方式（champions-cup模式）-1">6.2.5 ⭐ 机器人判断踢球队伍的方式（Champions Cup模式）</h4>
<p>Goal Kick的特点是：<strong>球一定放在某支队伍防守的球门区角落，该队伍获得踢球权</strong>。</p>
<p>机器人可以通过<strong>球的位置</strong>和<strong>球所在的半场</strong>来判断：</p>
<p><strong>判断逻辑</strong>：</p>
<pre><code>Goal Kick规则：
- 球放在【防守方】的球门区角落
- 【防守方】获得踢球权
- 因此：球在哪个半场的球门区，哪支队伍踢球
</code></pre>
<p><strong>B-Human 机器人端代码</strong>：</p>
<p><strong>文件位置</strong>: <code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code></p>
<p><strong>第999-1008行 - 通过球位置判断Goal Kick踢球队伍</strong>：</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">else</span>  <span class="token keyword">if</span><span class="token punctuation">(</span>gameControllerData<span class="token punctuation">.</span>setPlay <span class="token operator">==</span> SET_PLAY_GOAL_KICK<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>isKickingTeamValid<span class="token punctuation">)</span> <span class="token comment">// kickingTeam == 255，需要自己判断</span>
	<span class="token punctuation">{</span>
		<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>gameStateOverridden <span class="token operator">&amp;&amp;</span> theTeamBallModel<span class="token punctuation">.</span>isValid<span class="token punctuation">)</span>
<span class="token comment">// 关键判断：球的x坐标 &lt; 0 表示球在我方半场（我方防守的球门区）</span>
<span class="token comment">// 因此我方踢球 (isKickingTeam = true)</span>
			isKickingTeam <span class="token operator">=</span> theTeamBallModel<span class="token punctuation">.</span>position<span class="token punctuation">.</span><span class="token function">x</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> <span class="token number">0</span><span class="token punctuation">;</span>
		<span class="token keyword">else</span>  <span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>GameState<span class="token operator">::</span><span class="token function">isGoalKick</span><span class="token punctuation">(</span>lastGameControllerState<span class="token punctuation">)</span><span class="token punctuation">)</span>
			isKickingTeam <span class="token operator">=</span> <span class="token boolean">false</span><span class="token punctuation">;</span> <span class="token comment">// 无法判断时默认对方踢球</span>
<span class="token punctuation">}</span>
	<span class="token keyword">return</span> isKickingTeam <span class="token operator">?</span> GameState<span class="token operator">::</span>ownGoalKick <span class="token operator">:</span> GameState<span class="token operator">::</span>opponentGoalKick<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>代码解释</strong>：</p>
<ol>
<li>机器人收到 <code>setPlay = SET_PLAY_GOAL_KICK</code> 且 <code>kickingTeam = 255</code></li>
<li>检查 <code>theTeamBallModel.isValid</code>（球位置是否有效）</li>
<li><strong>关键判断</strong>：<code>theTeamBallModel.position.x() &lt; 0</code></li>
</ol>
<ul>
<li><code>x &lt; 0</code>：球在我方半场（我方防守的球门区）→ <strong>我方踢球</strong></li>
<li><code>x &gt; 0</code>：球在对方半场（对方防守的球门区）→ <strong>对方踢球</strong></li>
</ul>
<ol start="4">
<li>根据判断结果返回 <code>ownGoalKick</code> 或 <code>opponentGoalKick</code></li>
</ol>
<p><strong>坐标系说明</strong>：</p>
<pre><code>
B-Human坐标系（从我方视角）：
		 对方球门
			↑
	 ───────┼─────── x &gt; 0 (对方半场)
			│
	 ───────┼─────── x = 0 (中线)
			│
	 ───────┼─────── x &lt; 0 (我方半场)
			↓
		 我方球门
</code></pre>
<p><strong>判断流程图</strong>：</p>
<pre><code>		机器人收到 setPlay=1 (Goal Kick), kickingTeam=255
							↓
				检查 theTeamBallModel.isValid
							↓
			获取球位置 theTeamBallModel.position.x()
							↓
				 ┌──────────┴──────────┐
				 ↓ 					   ↓
				x &lt; 0 				 x &gt; 0
			(球在我方半场) 	      (球在对方半场)
				 ↓ 					   ↓
		isKickingTeam=true 		isKickingTeam=false
			 (我方踢球) 				(对方踢球)
				 ↓ 					   ↓
		返回 ownGoalKick 		返回 opponentGoalKick
</code></pre>
<p><strong>示例场景</strong>：</p>
<pre><code>假设：我方防守左球门（x&lt;0的一侧），对方防守右球门（x&gt;0的一侧）
场景：对方进攻，将球踢出我方的球门线
结果：
- 裁判喊 "Goal Kick"
- 球放在我方半场的球门区角落
- theTeamBallModel.position.x() &lt; 0
- isKickingTeam = true → 我方踢球
- 返回 GameState::ownGoalKick
</code></pre>
<hr>
<h3 id="corner-kick（角球）——-根据球位置判断">6.3 Corner Kick（角球）—— 根据球位置判断</h3>
<h4 id="触发条件-2">6.3.1 触发条件</h4>
<p>球越过球门线（goal line），且最后触球的是<strong>防守方</strong>（即守门员或后卫将球踢出自己的球门线）。</p>
<h4 id="球的位置-2">6.3.2 球的位置</h4>
<p>裁判将球放在<strong>角球点</strong>（corner），即球门线与边线的交点。</p>
<h4 id="gc操作员操作流程-2">6.3.3 GC操作员操作流程</h4>
<ol>
<li>裁判观察到防守方将球踢出球门线，喊出 “Corner Kick &lt;颜色&gt;”（如 “Corner Kick blue”）</li>
<li>GC操作员点击**进攻方（获得踢球权）**下方的 “Corner Kick” 按钮</li>
<li>裁判将球放在角球点</li>
</ol>
<h4 id="机器人收到的数据包-2">6.3.4 机器人收到的数据包</h4>

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
</table><h4 id="⭐-机器人判断踢球队伍的方式（champions-cup模式）-2">6.3.5 ⭐ 机器人判断踢球队伍的方式（Champions Cup模式）</h4>
<p>Corner Kick的特点是：<strong>球放在某支队伍防守的半场角落，但由对方（进攻方）踢球</strong>。</p>
<p>这与Goal Kick相反：</p>
<ul>
<li>Goal Kick：球在哪个半场，<strong>该半场的防守队伍</strong>踢球</li>
<li>Corner Kick：球在哪个半场，**对方队伍（进攻方）**踢球</li>
</ul>
<p><strong>判断逻辑</strong>：</p>
<pre><code>
Corner Kick规则：
- 球放在【防守方】的半场角落
- 【进攻方】获得踢球权
- 因此：球在哪个半场的角落，【另一支队伍】踢球
</code></pre>
<p><strong>B-Human 机器人端实际代码</strong>：</p>
<p><strong>文件位置</strong>: <code>Src/Modules/Infrastructure/GameStateProvider/GameStateProvider.cpp</code></p>
<p><strong>第1009-1019行 - 通过球位置判断Corner Kick踢球队伍</strong>：</p>
<pre class=" language-cpp"><code class="prism  language-cpp">
<span class="token keyword">else</span>  <span class="token keyword">if</span><span class="token punctuation">(</span>gameControllerData<span class="token punctuation">.</span>setPlay <span class="token operator">==</span> SET_PLAY_CORNER_KICK<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>isKickingTeamValid<span class="token punctuation">)</span> <span class="token comment">// kickingTeam == 255，需要自己判断</span>
	<span class="token punctuation">{</span>
		<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>gameStateOverridden <span class="token operator">&amp;&amp;</span> theTeamBallModel<span class="token punctuation">.</span>isValid<span class="token punctuation">)</span>
	<span class="token comment">// 关键判断：球的x坐标 &gt; 0 表示球在对方半场（对方防守的角落）</span>
	<span class="token comment">// Corner Kick由进攻方踢，所以球在对方半场时我方踢球</span>
		isKickingTeam <span class="token operator">=</span> theTeamBallModel<span class="token punctuation">.</span>position<span class="token punctuation">.</span><span class="token function">x</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">;</span>
		<span class="token keyword">else</span>  <span class="token keyword">if</span><span class="token punctuation">(</span><span class="token operator">!</span>GameState<span class="token operator">::</span><span class="token function">isCornerKick</span><span class="token punctuation">(</span>lastGameControllerState<span class="token punctuation">)</span><span class="token punctuation">)</span>
		 isKickingTeam <span class="token operator">=</span> <span class="token boolean">false</span><span class="token punctuation">;</span> <span class="token comment">// 无法判断时默认对方踢球</span>
<span class="token punctuation">}</span>
	<span class="token keyword">return</span> isKickingTeam <span class="token operator">?</span> GameState<span class="token operator">::</span>ownCornerKick <span class="token operator">:</span> GameState<span class="token operator">::</span>opponentCornerKick<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>代码解释</strong>：</p>
<ol>
<li>机器人收到 <code>setPlay = SET_PLAY_CORNER_KICK</code> 且 <code>kickingTeam = 255</code></li>
<li>检查 <code>theTeamBallModel.isValid</code>（球位置是否有效）</li>
<li><strong>关键判断</strong>：<code>theTeamBallModel.position.x() &gt; 0</code>（注意：与Goal Kick相反！）</li>
</ol>
<ul>
<li><code>x &gt; 0</code>：球在对方半场（对方防守的角落）→ <strong>我方踢球</strong>（我方是进攻方）</li>
<li><code>x &lt; 0</code>：球在我方半场（我方防守的角落）→ <strong>对方踢球</strong>（对方是进攻方）</li>
</ul>
<ol start="4">
<li>根据判断结果返回 <code>ownCornerKick</code> 或 <code>opponentCornerKick</code></li>
</ol>
<p><strong>Goal Kick vs Corner Kick 判断对比</strong>：</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">// Goal Kick: 球在我方半场 → 我方踢球（防守方踢）</span>
isKickingTeam <span class="token operator">=</span> theTeamBallModel<span class="token punctuation">.</span>position<span class="token punctuation">.</span><span class="token function">x</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> <span class="token number">0</span>
  
<span class="token comment">// Corner Kick: 球在对方半场 → 我方踢球（进攻方踢）</span>

isKickingTeam <span class="token operator">=</span> theTeamBallModel<span class="token punctuation">.</span>position<span class="token punctuation">.</span><span class="token function">x</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">;</span>
</code></pre>
<p><strong>判断流程图</strong>：</p>
<pre><code>		机器人收到 setPlay=3 (Corner Kick), kickingTeam=255
								↓
					检查 theTeamBallModel.isValid
								↓
				获取球位置 theTeamBallModel.position.x()
								↓
					 ┌──────────┴──────────┐
					 ↓ 					   ↓
					x &gt; 0 				 x &lt; 0
				(球在对方半场角落) 	(球在我方半场角落)
					 ↓ 						↓
			isKickingTeam=true 		isKickingTeam=false
			(我方踢球-进攻方) 		(对方踢球-进攻方)
					↓ 						↓
			返回 ownCornerKick  		返回 opponentCornerKick

</code></pre>
<p><strong>示例场景</strong>：</p>
<pre><code>假设：我方防守左球门（x&lt;0的一侧），对方防守右球门（x&gt;0的一侧）
场景：对方守门员将球踢出自己的球门线（右侧）
结果：
- 裁判喊 "Corner Kick"
- 球放在对方半场的角球点
- theTeamBallModel.position.x() &gt; 0
- isKickingTeam = true → 我方踢球（我方是进攻方）
- 返回 GameState::ownCornerKick
</code></pre>
<p><strong>三种出界状态判断方式总结</strong>：</p>

<table>
<thead>
<tr>
<th>出界类型</th>
<th>判断方式</th>
<th>关键代码</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Kick-in</strong></td>
<td>识别裁判手势</td>
<td><code>checkForRefereeSignal(RefereeSignal::kickInLeft/Right)</code></td>
</tr>
<tr>
<td><strong>Goal Kick</strong></td>
<td>球位置 x &lt; 0 → 我方踢</td>
<td><code>isKickingTeam = theTeamBallModel.position.x() &lt; 0</code></td>
</tr>
<tr>
<td><strong>Corner Kick</strong></td>
<td>球位置 x &gt; 0 → 我方踢</td>
<td><code>isKickingTeam = theTeamBallModel.position.x() &gt; 0</code></td>
</tr>
</tbody>
</table><hr>
<h2 id="附录：代码流程图">附录：代码流程图</h2>
<pre><code>	GC操作员点击 Goal Kick/Kick-in/Corner Kick 按钮
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
需要识别裁判手势 					    无需额外操作
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
