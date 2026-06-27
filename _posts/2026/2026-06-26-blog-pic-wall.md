---
title: 博客增加图墙
description: 目前只有博主手绘
date: 2026-06-26
categories:
  - blog
tags:
  - blog
author: blackbzy
update_date: false
pin: false
toc: true
comments: true
image:
  path: blog/pic-wall.png
  alt: 效果如图
---

> 算是个人版爱好展示把，有回应可能博主更的勤快点(╹ڡ╹ )
{: .prompt-info }

基于我这个博客的架构，我想加一个图墙页面，然后我图片存在cf的r2，能自动获取OSS存储的图片然后生成页面，我用博客添加链接去引用。

基于这个思路，我开始了下面的改造。

## 第一步：在 Cloudflare 部署专属 Workers 后端
- 登录 Cloudflare 后台，点击左侧的 **Workers 和 Pages (Workers & Pages)** -> **创建应用程序 (Create Application)** -> **创建 Worker**。
- 名字可以叫 `r2-gallery-api`，直接点击部署。
- 点击 **编辑代码 (Edit Code)**，将里面的默认代码清空，替换为以下精简高效的 API 逻辑：

```js
export default {

  async fetch(request, env) {

    // 处理跨域请求 (CORS)，允许你的博客主站无障碍拉取

    const corsHeaders = {

      "Access-Control-Allow-Origin": "*",

      "Access-Control-Allow-Methods": "GET, OPTIONS",

      "Access-Control-Allow-Headers": "Content-Type",

      "Content-Type": "application/json;charset=UTF-8",

    };

  

    if (request.method === "OPTIONS") {

      return new Response(null, { headers: corsHeaders });

    }

  

    try {

      // 1. 列出 R2 桶内的所有对象（这里可以根据需求限制数量或设置前缀）

      const options = { limit: 500 };

      const listed = await env.MY_BUCKET.list(options);

  

      // 2. 筛选出常见的图片格式，并组装成完整的外链 URL

      // 请将下面的 https://pub-xxxxxx.r2.dev 替换为你 R2 桶绑定的自定义域名或公共域名

      const r2PublicUrl = "https://pub-xxxxxx.r2.dev";

      const images = listed.objects

        .filter(obj => /\.(jpg|jpeg|png|gif|webp|avif)$/i.test(obj.key))

        .map(obj => ({

          url: `${r2PublicUrl}/${obj.key}`,

          key: obj.key,

          uploaded: obj.uploaded

        }));

  

      // 按上传时间倒序排列，确保新照片永远在最前面

      images.sort((a, b) => new Date(b.uploaded) - new Date(a.uploaded));

  

      return new Response(JSON.stringify(images), { headers: corsHeaders });

    } catch (e) {

      return new Response(JSON.stringify({ error: e.message }), {

        status: 500,

        headers: corsHeaders,

      });

    }

  },

};

```

- 点击右上角的 **部署 (Deploy)**。
- **最关键的一步（绑定R2）：** 返回该 Worker 的主页面，点击 **设置 (Settings)** -> **变量 (Variables)**。向下滚动找到 **R2 存储桶绑定 (R2 Bucket Bindings)**，点击 **添加绑定 (Add Binding)**：
    - **变量名称 (Variable name):** 必须填写 **`MY_BUCKET`**（要和代码中的 `env.MY_BUCKET` 一致）。
    - **R2 存储桶 (R2 bucket):** 选择你存放图片的那个 R2 存储桶。
    - 点击保存。现在你的 Worker 就能安全、实时地读取 R2 了。它会给你一个类似 `https://r2-gallery-api.xxxx.workers.dev` 的公共 API 地址。


## 第二步：在 Jekyll 博客中创建实时前端页面
博客源码 `pages/` 目录下（或者你喜欢的位置）新建一个 `gallery.html`，这是纯前端异步渲染的页面，完美适配 Chirpy 主题。

```html
---
layout: page
title: 杂草丛中 纸上余温
icon: fas fa-palette
permalink: /gallery/
order: 5
---

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@fancyapps/ui@5.0/dist/fancybox/fancybox.css" />
<script src="https://cdn.jsdelivr.net/npm/@fancyapps/ui@5.0/dist/fancybox/fancybox.umd.js"></script>

<style>
  .gallery-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); grid-gap: 15px; padding: 10px 0; }
  .gallery-item { background-color: var(--card-bg); border-radius: 8px; overflow: hidden; box-shadow: 0 4px 6px rgba(0,0,0,0.05); transition: transform 0.2s ease; height: 200px; } /* 固定高度让瀑布流排列更整齐 */
  .gallery-item:hover { transform: translateY(-4px); }
  .gallery-link { position: relative; display: block; width: 100%; height: 100%; }
  .gallery-copyright { position: absolute; bottom: 0; left: 0; right: 0; background: linear-gradient(transparent, rgba(0,0,0,0.6)); color: #fff; font-size: 11px; padding: 15px 10px 5px 10px; opacity: 0; transition: opacity 0.3s ease; pointer-events: none; }
  .gallery-item:hover .gallery-copyright { opacity: 1; }
  #gallery-loading { text-align: center; padding: 3rem; color: var(--text-muted); }

  /* 加载更多按钮样式 */
  .load-more-wrapper { text-align: center; margin: 30px 0; }
  .btn-load-more { background: var(--card-bg); color: var(--text-color); border: 1px solid var(--border-color); padding: 10px 24px; border-radius: 20px; cursor: pointer; font-size: 14px; transition: all 0.2s; }
  .btn-load-more:hover { background: var(--border-color); transform: scale(1.03); }

  /* 🛠️ 终极自适应：侧边栏排版与防止图片挤压 */
  @media (min-width: 992px) {
    .fancybox__container { display: flex !important; flex-direction: row !important; }
    /* 强行限制轮播区宽度，预留 380px 给侧边栏 */
    .fancybox__carousel { width: calc(100% - 380px) !important; flex: none !important; }
    /* 强行控制大图的最大高宽，防止横向或纵向巨图把侧边栏顶开 */
    .fancybox__slide .fancybox__content { max-width: 100% !important; max-height: 90vh !important; object-fit: contain !important; }

    .fancybox-dynamic-sidebar {
      width: 380px; height: 100%; flex-shrink: 0;
      background: var(--main-bg) !important;
      color: var(--text-color) !important;
      border-left: 1px solid var(--border-color);
      overflow-y: auto; padding: 20px; z-index: 99;
      box-shadow: -4px 0 15px rgba(0,0,0,0.05);
    }
  }
  @media (max-width: 991px) {
    .fancybox__container { display: flex !important; flex-direction: column !important; }
    .fancybox-dynamic-sidebar {
      width: 100%; max-height: 40vh; flex-shrink: 0;
      background: var(--main-bg) !important;
      color: var(--text-color) !important;
      overflow-y: auto; padding: 15px; z-index: 99;
    }
  }

  .fancybox-dynamic-sidebar h4 { margin-top: 0; font-size: 15px; font-weight: 600; color: var(--heading-color); border-bottom: 1px solid var(--border-color); padding-bottom: 10px; margin-bottom: 15px;}
  .fancybox-dynamic-sidebar .waline-container { min-height: 200px; }
</style>

<div id="gallery-loading">📸 正在从银河系边缘同步最新照片...</div>
<div class="gallery-grid" id="gallery-container" style="display: none;"></div>
<div class="load-more-wrapper" id="load-more-box" style="display: none;">
  <button class="btn-load-more" id="btn-load-more">查看更多美好 ✨</button>
</div>

<script type="module">
  // 1. 全局捕获无意义的请求中止报错，保护控制台干净
  window.addEventListener('unhandledrejection', (event) => {
    if (event.reason && (
      event.reason.name === 'AbortError' ||
      event.reason.message?.includes('signal is aborted') ||
      event.reason.message?.includes('Cancel')
    )) {
      event.preventDefault();
      console.log('Ignored formal AbortError from Waline.');
    }
  });

  let galleryWalineInstance = null;
  let currentLoadTimeout = null;
  const walineServerURL = 'https://comments.doman.com';

  // 分页控制核心变量
  let allImages = [];
  let currentIndex = 0;
  const pageSize = 10;

  if (!document.getElementById('waline-style')) {
    const link = document.createElement('link');
    link.id = 'waline-style';
    link.rel = 'stylesheet';
    link.href = 'https://unpkg.com/@waline/client@v3/dist/waline.css';
    document.head.appendChild(link);
  }

  // 渲染一组图片的通用函数
  function renderNextPage() {
    const container = document.getElementById("gallery-container");
    const loadMoreBox = document.getElementById("load-more-box");

    const nextGroup = allImages.slice(currentIndex, currentIndex + pageSize);

    nextGroup.forEach(img => {
      const item = document.createElement("div");
      item.className = "gallery-item";
      const photoKey = `/gallery/${img.key}`;

      item.innerHTML = `
        <a href="${img.url}" data-fancybox="gallery" data-caption="${photoKey}" class="gallery-link"
           style="display: block; width: 100%; height: 100%; background: url('${img.url}') center center / cover no-repeat;">
          <div class="gallery-copyright">
             <span>© blackbzy</span>
          </div>
        </a>
      `;
      container.appendChild(item);
    });

    currentIndex += nextGroup.length;

    // 如果所有照片全加完了，藏掉按钮
    if (currentIndex >= allImages.length) {
      loadMoreBox.style.display = "none";
    } else {
      loadMoreBox.style.display = "block";
    }
  }

  document.addEventListener("DOMContentLoaded", async () => {
    const apiUrl = "换成你刚刚创建的 Cloudflare Worker 的实际 API 地址";
    const container = document.getElementById("gallery-container");
    const loading = document.getElementById("gallery-loading");
    const btnLoadMore = document.getElementById("btn-load-more");

    // 绑定“加载更多”按钮事件
    btnLoadMore.addEventListener("click", renderNextPage);

    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error("API 响应异常");
      allImages = await response.json();

      if (allImages.length === 0) {
        loading.innerText = "🔍 存储桶里还没有照片哦";
        return;
      }

      loading.style.display = "none";
      container.style.display = "grid";

      // 首次只加载第一页（10张）
      renderNextPage();

      // 初始化绑定具有强约束力的 Fancybox 布局
      Fancybox.bind("[data-fancybox='gallery']", {
        Hash: false,
        Thumbs: { autoStart: false },
        on: {
          initLayout: (fancybox) => {
            const sidebarHtml = `
              <div class="fancybox-dynamic-sidebar">
                <h4 id="dynamic-photo-title">💬 照片评论</h4>
                <div id="dynamic-waline-box" class="waline-container"></div>
              </div>
            `;
            fancybox.container.insertAdjacentHTML("beforeend", sidebarHtml);
          },

          done: (fancybox, slide) => {
            const currentPhotoPath = slide.triggerEl.getAttribute("data-caption");
            const photoName = currentPhotoPath.split('/').pop();

            const titleEl = fancybox.container.querySelector("#dynamic-photo-title");
            const walineEl = fancybox.container.querySelector("#dynamic-waline-box");

            if (titleEl) {
              titleEl.innerHTML = `💬 照片评论 (${photoName})`;
            }

            if (currentLoadTimeout) {
              clearTimeout(currentLoadTimeout);
              currentLoadTimeout = null;
            }

            if (galleryWalineInstance) {
              try {
                galleryWalineInstance.destroy();
              } catch(e) { }
              galleryWalineInstance = null;
            }
            if (walineEl) {
              walineEl.innerHTML = '';
            }

            if (!walineEl) return;

            currentLoadTimeout = setTimeout(() => {
              import('https://unpkg.com/@waline/client@v3/dist/waline.js')
                .then((module) => {
                  const { init } = module;

                  if (typeof init === 'function' && walineEl) {
                    walineEl.innerHTML = '';

                    galleryWalineInstance = init({
                      el: walineEl,
                      serverURL: walineServerURL,
                      path: currentPhotoPath,
                      dark: 'html[data-mode="dark"]',
                      reaction: true,
                      requiredMeta: ['nick'],
                      emoji: [
                        '//unpkg.com/@waline/emojis@1.4.0/weibo',
                        '//unpkg.com/@waline/emojis@1.4.0/bmoji'
                      ]
                    });
                  }
                })
                .catch(err => {
                  console.error('图墙 Waline 模块加载失败:', err);
                });
            }, 80);
          },

          close: () => {
            if (currentLoadTimeout) {
              clearTimeout(currentLoadTimeout);
              currentLoadTimeout = null;
            }
            if (galleryWalineInstance) {
              try {
                galleryWalineInstance.destroy();
              } catch(e) { }
              galleryWalineInstance = null;
            }
          }
        }
      });

    } catch (err) {
      console.error(err);
      loading.innerText = "❌ 无法加载图墙，请检查网络或配置";
    }
  });
</script>

```


## 最后图片的批量处理脚本，方便上传图片的管理
watermark.py

```py

import os
from datetime import datetime
from PIL import Image
from PIL import Image, ImageOps

# ==================== 配置区域 ====================
SOURCE_DIR = "D:/a-blog/pic-sketch"      # 原始照片目录
OUTPUT_DIR = "D:/a-blog/pic-sketch-water"     # 加完水印和改好名的输出目录
LOGO_PATH = "D:/a-blog专用/视觉资产使用手册/shuiyin.png"         # 你的水印图片路径（建议是透明背景的 PNG）
# ==================================================

if not os.path.exists(OUTPUT_DIR):
    os.makedirs(OUTPUT_DIR)

def batch_process():
    if not os.path.exists(SOURCE_DIR):
        os.makedirs(SOURCE_DIR)
        print(f"⚠️ 首次运行：已自动创建【{SOURCE_DIR}】文件夹，请放入原图后重新运行。")
        return

    if not os.path.exists(LOGO_PATH):
        print(f"❌ 错误：在当前目录下找不到水印图片【{LOGO_PATH}】！")
        return

    files = [f for f in os.listdir(SOURCE_DIR) if f.lower().endswith(('.png', '.jpg', '.jpeg', '.webp'))]
    if not files:
        print(f"ℹ️ 提示：【{SOURCE_DIR}】文件夹里没有找到任何照片。")
        return

    logo_org = Image.open(LOGO_PATH).convert("RGBA")
    count = 1

    print("🚀 开始进行【高压缩比 + 水印】批量处理...")
    for filename in sorted(files):
        img_path = os.path.join(SOURCE_DIR, filename)
        try:
            with Image.open(img_path) as img:
				# ====== 核心修复：自动根据 Exif 标签旋转图片像素 ======
                img = ImageOps.exif_transpose(img) 
                # =====================================================
                # 1. 批量命名统一换成扩展名 .webp，更加现代化
                date_str = datetime.now().strftime("%Y%m%d")
                new_name = f"gallery_{date_str}_{count:03d}.webp"
                
                # 2. 【核心优化】尺寸大图裁剪
                # 如果单张照片分辨率宽度超过 2000px，自动等比缩小到 2000px 宽，对于网页大屏展示绰绰有余
                img_w, img_h = img.size
                max_width = 2000
                if img_w > max_width:
                    ratio = max_width / img_w
                    new_w = max_width
                    new_h = int(img_h * ratio)
                    # 使用高保真 LANCZOS 算法缩放
                    img = img.resize((new_w, new_h), Image.Resampling.LANCZOS)
                    img_w, img_h = new_w, new_h
                
                img_rgba = img.convert("RGBA")
                
                # 3. 动态调整水印大小：保持占大图宽度的 14%
                target_logo_w = int(img_w * 0.14)
                target_logo_h = int(logo_org.size[1] * (target_logo_w / logo_org.size[0]))
                logo_resized = logo_org.resize((target_logo_w, target_logo_h), Image.Resampling.LANCZOS)
                
                # 右下角留出 3% 边距
                x = img_w - target_logo_w - int(img_w * 0.03)
                y = img_h - target_logo_h - int(img_h * 0.03)
                
                # 4. 压制水印
                watermark_layer = Image.new("RGBA", img_rgba.size, (0, 0, 0, 0))
                watermark_layer.paste(logo_resized, (x, y))
                result_rgba = Image.alpha_composite(img_rgba, watermark_layer)
                
                # 5. 【核心优化】有损 WebP 强力压缩
                # quality=75 是业界公认的黄金分割点，体积缩减 70%-80%，画质肉眼无差别
                output_path = os.path.join(OUTPUT_DIR, new_name)
                result_rgba.save(output_path, "WEBP", quality=75, method=6)
                
                # 计算压缩前后的体积对比
                orig_size = os.path.getsize(img_path) / 1024 / 1024
                new_size = os.path.getsize(output_path) / 1024
                print(f"✅ 成功: {filename} ({orig_size:.1f}MB) -> {new_name} ({new_size:.0f}KB)")
                count += 1
        except Exception as e:
            print(f"❌ 处理失败 {filename}: {e}")
            
    print(f"\n🎉 搞定！所有高效压缩后的 WebP 照片已放入【{OUTPUT_DIR}】。")

if __name__ == "__main__":
    try:
        batch_process()
    finally:
        print("\n" + "="*40)
        os.system("pause")

```


---
故事未完:177
**Thoughts**:: justdoit.
