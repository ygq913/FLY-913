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
</ul>
</li>
</ul>

    </div>
  </div>
  <div class="stackedit__right">
    <div class="stackedit__html">
      <blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>
<h2 id="gamecontroller3-完整指南">GameController3 完整指南</h2>
<p><strong>目录</strong><br>
1.获取并编辑GameController3<br>
2.2025规则更新要点<br>
3.GC模式选择说明<br>
4.比赛流程详解<br>
5.基于gc更新的关键代码解析<br>
6.三种出界状态详解</p>
<p><strong>一、获取并编辑GameController3：</strong><br>
<strong>1.1获取源码：</strong></p>
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
<p><strong>二、2025规则更新要点(部分)</strong><br>
根据2025-6月发布的规则文件，主要变更如下（<strong>该部分只适用2025年更新规则</strong>）：</p>
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
<p><strong>3.GC模式选择说明</strong><br>
| 模式 | 机器人数量 | hideKickingSide | 需要识别裁判手势？ | 适用场景 |</p>
<p>|------|----------------|--------------------|-------------------------|-------------|</p>
<p>| Champions Cup | 7人 | true | ✅ 需要 | 正式比赛(7v7) |</p>
<p>| Champions Cup 5 vs. 5 | 5人 | true | ✅ 需要 | 正式比赛(5v5) |</p>
<p>| Challenge Shield | 5人 | false | ❌ 不需要 | 新队伍/入门级 |</p>
<p>| Most Passes Leaderboard | 2人 | false | ❌ 不需要 | 技术挑战赛 |</p>
<p>1.GC操作员的视角：<br>
1&gt;裁判看到球出界，喊“kick-in blue”（即蓝队踢球）<br>
2&gt;GC操作员按下蓝方队伍下方的“kick-in”按钮<br>
3&gt;GC记录下蓝队踢球（用于日志、计时等）<br>
2.机器人受到的消息（Champios Cup模式）：<br>
由于‘hideKickingSide: true’,机器人收到的</p>

    </div>
  </div>
</body>

</html>
