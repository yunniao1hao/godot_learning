<details> 
<summary>如何使用Godot自己的噪声？</summary>
<p>DeepSeek：</p>
<p>在 Godot 中使用噪声纹理（NoiseTexture）并集成到着色器中，可以通过以下步骤实现。重点会放在如何通过检查器面板（Inspector）配置噪声纹理：</p>
<hr>
<h3>一、创建 NoiseTexture 资源</h3>
<ol>
<li><strong>右键项目资源窗口</strong>
在 Godot 的资源面板（FileSystem）中右键，选择 <strong>New Resource</strong>。</li>
<li><strong>选择 NoiseTexture</strong>
在资源类型搜索框中输入 <code>NoiseTexture</code>，创建该资源。</li>
<li><strong>配置噪声参数</strong>
<ul>
<li><strong>Noise</strong>：点击 <code>[empty]</code> 右侧的箭头，选择 <strong>New FastNoiseLite</strong>（Godot 4.x）或其他噪声类型（如 <code>SimplexNoise</code>）。</li>
<li><strong>Width/Height</strong>：设置噪声纹理的分辨率（如 <code>512x512</code>）。</li>
<li><strong>Seamless</strong>：勾选此项可使纹理无缝平铺。</li>
<li><strong>Generate Mipmaps</strong>：根据需要开启。</li>
</ul>
</li>
<li><strong>保存为独立资源</strong>
点击右上角 <strong>Save</strong> 按钮，将噪声纹理保存为 <code>.tres</code> 文件（如 <code>noise_texture.tres</code>）。</li>
</ol>
<hr>
<h3>二、在 ShaderMaterial 中绑定噪声纹理</h3>
<ol>
<li>
<p><strong>创建 ShaderMaterial</strong>
为你的材质节点（如 <code>MeshInstance</code>）添加 <code>ShaderMaterial</code>，并为其新建或关联一个着色器脚本（<code>.shader</code> 文件）。</p>
</li>
<li>
<p><strong>在着色器中声明 Uniform</strong>
在着色器代码中声明一个 <code>uniform sampler2D</code> 变量，例如：</p>
<p>glsl</p>
<p>复制</p>
<pre><code class="code-highlight"><span class="code-line line-number" line="1">// 片段着色器示例（Godot 4.0+）
</span><span class="code-line line-number" line="2">shader_type canvas_item; // 或 spatial
</span><span class="code-line line-number" line="3">
</span><span class="code-line line-number" line="4">uniform sampler2D noise_tex; // 噪声纹理
</span><span class="code-line line-number" line="5">
</span><span class="code-line line-number" line="6">void fragment() {
</span><span class="code-line line-number" line="7">    // 采样噪声纹理
</span><span class="code-line line-number" line="8">    float noise = texture(noise_tex, UV).r;
</span><span class="code-line line-number" line="9">    COLOR = vec4(vec3(noise), 1.0);
</span><span class="code-line line-number" line="10">}
</span></code></pre>
</li>
<li>
<p><strong>在检查器中绑定 NoiseTexture</strong></p>
<ul>
<li>在 <code>ShaderMaterial</code> 的 <strong>Shader Parameters</strong> 部分，找到 <code>noise_tex</code>。</li>
<li>点击右侧的 <code>[空]</code>，选择 <strong>Load</strong> 或直接拖入之前保存的 <code>noise_texture.tres</code>。</li>
</ul>
</li>
</ol>
<hr>
<h3>三、动态调整噪声参数</h3>
<p>在检查器中可直接修改 <code>NoiseTexture</code> 的属性：</p>
<ul>
<li><strong>Noise 配置</strong>：点击 <code>Noise</code> 属性，调整噪声类型（Perlin、Simplex 等）、<code>seed</code>、<code>frequency</code> 等参数。</li>
<li><strong>实时预览</strong>：修改参数后，Godot 会自动更新纹理效果，方便调试。</li>
</ul>
<hr>
<h3>四、在着色器中使用噪声</h3>
<p>采样噪声后，可将其用于多种效果，例如：</p>
<p>glsl</p>
<p>复制</p>
<pre><code class="code-highlight"><span class="code-line line-number" line="1">// 位移效果
</span><span class="code-line line-number" line="2">void vertex() {
</span><span class="code-line line-number" line="3">    float noise = texture(noise_tex, UV).r;
</span><span class="code-line line-number" line="4">    VERTEX.y += noise * 10.0; // 沿Y轴位移
</span><span class="code-line line-number" line="5">}
</span><span class="code-line line-number" line="6">
</span><span class="code-line line-number" line="7">// 溶解效果
</span><span class="code-line line-number" line="8">void fragment() {
</span><span class="code-line line-number" line="9">    float noise = texture(noise_tex, UV).r;
</span><span class="code-line line-number" line="10">    if (noise &lt; 0.5) discard; // 丢弃低于阈值的像素
</span><span class="code-line line-number" line="11">}
</span></code></pre>
<hr>
<h3>五、注意事项</h3>
<ol>
<li>
<p><strong>性能优化</strong></p>
<ul>
<li>大尺寸噪声纹理（如 <code>1024x1024</code>）会占用更多内存，建议根据需求平衡精度和性能。</li>
<li>使用 <code>seamless</code> 选项确保平铺时无接缝。</li>
</ul>
</li>
<li>
<p><strong>版本差异</strong></p>
<ul>
<li><strong>Godot 3.x</strong>：使用 <code>OpenSimplexNoise</code> + <code>NoiseTexture</code>。</li>
<li><strong>Godot 4.x</strong>：改用 <code>FastNoiseLite</code>，支持更多噪声类型（如 Cellular）。</li>
</ul>
</li>
<li>
<p><strong>动态生成噪声</strong>
可通过代码实时修改噪声参数：</p>
<p>gdscript</p>
<p>复制</p>
<pre><code class="code-highlight"><span class="code-line line-number" line="1">var noise = FastNoiseLite.new()
</span><span class="code-line line-number" line="2">noise.seed = randi()
</span><span class="code-line line-number" line="3">$ShaderMaterial.get_shader_parameter("noise_tex").noise = noise
</span></code></pre>
</li>
</ol>
<hr>
<p>通过以上步骤，你可以高效地在 Godot 的着色器中集成噪声纹理，并通过检查器灵活调整参数。</p>s
</details>

