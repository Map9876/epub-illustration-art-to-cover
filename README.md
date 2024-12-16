# epub-illustration-art-to-cover
epub轻小说修改图片到前面几页

## epub修改图片到首页，以及格式介绍

```
import zipfile
import os
import tempfile
import shutil

def find_oebps_folder(first_dir):
   for root, dirs, files in os.walk(first_dir):
        for file in files:
            print(file)
            if file.lower().endswith('.opf'):
                return root
   return None

def find_opf_file(directory):
    print(directory)
    for root, dirs, files in os.walk(directory):
        for file in files:
            print(file)
            if file.endswith('.opf'):
                return os.path.join(root, file)
        return None

def find_image_folder(directory):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.lower().endswith(('.jpg', '.jpeg', '.png', '.gif')):
                return root
    return None

def find_xhtml_folder(directory):
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith('.xhtml'):
                return root
    return None

import os

def print_file_tree(directory, prefix=''):
    if not os.path.isdir(directory):
        print("提供的路径不是一个文件夹")
        return

    files = os.listdir(directory)
    for i, file in enumerate(files):
        print(prefix + '├── ' + file)
        path = os.path.join(directory, file)
        if os.path.isdir(path):
            if i == len(files) - 1:
                # 最后一个文件
                new_prefix = prefix + '    '
            else:
                new_prefix = prefix + '|   '
            print_file_tree(path, new_prefix)

# 假设temp_dir是之前解压EPUB文件到的临时文件夹的路径


import os
import tempfile
import shutil

def modify_epub(epub_path):
    # 创建一个临时文件夹
    temp_dir = tempfile.mkdtemp()
    
    # 解压EPUB文件到临时文件夹
    with zipfile.ZipFile(epub_path, 'r') as epub_zip:
        epub_zip.extractall(temp_dir)
    print_file_tree(temp_dir)
    # 获取OEBPS文件夹路径
   # oebps_folder = os.path.join(temp_dir, 'OEBPS')
    
    oebps_folder = find_oebps_folder(temp_dir)
    if not oebps_folder:
        print("没有找到oebps文件夹。")
        return

    # 查找opf文件
    content_opf_path = find_opf_file(oebps_folder)
    if not content_opf_path:
        print("没有找到opf文件。")
        return
    
    # 查找图片文件夹
    images_folder = find_image_folder(oebps_folder)
    if not images_folder:
        print("没有找到图片文件夹。")
        return
    
    # 查找XHTML文件夹
    xhtml_folder = find_xhtml_folder(oebps_folder)
    if not xhtml_folder:
        print("没有找到XHTML文件夹。")
        return
    
    # 获取Images文件夹中的所有图片
    images = [f for f in os.listdir(images_folder) if f.lower().endswith(('.jpg', '.jpeg', '.png', '.gif'))]
    
    # 创建一个新的XHTML文件内容
    xhtml_content = f"""<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Image Gallery</title>
</head>
<body>"""
    
    # 添加图片到XHTML内容
    for image in images:
        xhtml_content += f'<div class="duokan-image-single illus"><img alt="{os.path.splitext(image)[0]}" src="{os.path.relpath(os.path.join(images_folder, image), xhtml_folder)}" /></div>\n'
    
    xhtml_content += """</body>
</html>"""
    
    # 新的XHTML文件路径
    new_xhtml_path = os.path.join(xhtml_folder, 'aaa.xhtml')
    
    # 写入新的XHTML文件
    with open(new_xhtml_path, 'w', encoding='utf-8') as xhtml_file:
        xhtml_file.write(xhtml_content)
    
    # 读取并更新content.opf文件
    with open(content_opf_path, 'r', encoding='utf-8') as opf_file:
        content_opf = opf_file.read()
    
    # 更新manifest
    manifest_start = content_opf.find('<manifest>')
    manifest_end = content_opf.find('</manifest>')
    manifest = content_opf[manifest_start:manifest_end + len('</manifest>')]
    new_item = f'<item id="aaa" href="{os.path.relpath(new_xhtml_path, oebps_folder)}" media-type="application/xhtml+xml"/>'
    updated_manifest = manifest.replace('</manifest>', new_item + '</manifest>')
    
    # 更新spine
    spine_start = content_opf.find('<spine toc="ncx">')
    spine_end = content_opf.find('</spine>')
    spine = content_opf[spine_start:spine_end + len('</spine>')]
    new_itemref = '<itemref idref="aaa" />'
    updated_spine = spine.replace('<spine toc="ncx">', f'<spine toc="ncx">{new_itemref}')
    
    # 更新content.opf文件
    updated_content_opf = content_opf.replace(manifest, updated_manifest)
    updated_content_opf = updated_content_opf.replace(spine, updated_spine)
    
    # 写入更新后的content.opf文件
    with open(content_opf_path, 'w', encoding='utf-8') as opf_file:
        opf_file.write(updated_content_opf)
    
    # 重新压缩修改后的文件回EPUB
    with zipfile.ZipFile(epub_path, 'w', zipfile.ZIP_DEFLATED) as epub_zip:
        for root, dirs, files in os.walk(temp_dir):
            for file in files:
                file_path = os.path.join(root, file)
                epub_zip.write(file_path, file_path[len(temp_dir):])
    print("ok")
    # 清理临时文件夹
    shutil.rmtree(temp_dir)

# 使用示例
epub_path = '/content/01和班.epub'
modify_epub(epub_path)
```

## 使用示例，代码末尾添加以下两行 :
```

epub_path = '/content/01和班.epub'
modify_epub(epub_path)
```
epub本质上就是zip压缩文件，修改为zip后缀或者直接解压都能打开

安装tree命令，用tree命令打印出比较可视的文件夹列表 :

pkg install tree

对于一个  `海空りく - 落第騎士英雄譚 01.epub` → `海空りく - 落第騎士英雄譚 01.zip` → `海空りく - 落第騎士英雄譚 01`文件夹
```
~ $ tree "/storage/emulated/0/Download/Browser/海空りく - 落第騎士英雄譚 01" # 这里是命令，tree 跟着输入解压后的文件夹
/storage/emulated/0/Download/Browser/海空りく - 落第 騎士英雄譚 01
├── META-INF
│   └── container.xml
├── OEBPS
│   ├── Images
│   │   ├── 000.jpg
│   │   ├── 001.jpg
│   │   ├── 002.jpg
│   │   ├── 003.jpg
│   │   ├── 004.jpg
│   │   ├── 005.jpg
│   │   ├── 006.jpg
│   │   ├── 007.jpg
│   │   ├── 008.jpg
│   │   ├── 009.jpg
│   │   ├── 010.jpg
│   │   ├── 011.jpg
│   │   ├── 012.jpg
│   │   ├── 013.jpg
│   │   ├── 014.jpg
│   │   ├── 015.jpg
│   │   ├── 016.jpg
│   │   ├── 017.jpg
│   │   └── 018.jpg
│   ├── Text
│   │   ├── Section0001.xhtml
│   │   ├── Section0002.xhtml
│   │   ├── Section0003.xhtml
│   │   ├── Section0004.xhtml
│   │   ├── Section0005.xhtml
│   │   ├── Section0006.xhtml
│   │   ├── Section0007.xhtml
│   │   ├── Section0008.xhtml
│   │   ├── Section0009.xhtml
│   │   └── cover.xhtml
│   ├── content.opf
│   └── toc.ncx
└── mimetype

5 directories, 33 files
```

看到就是一些图片，和/OEBPS/Text/文件夹下.xhtml的文本文件，.xhtml里面就是轻小说正文文本

/OEBPS/content.opf内文本如下 :

```
<?xml version="1.0"  encoding="UTF-8"?>
<package xmlns="http://www.idpf.org/2007/opf" unique-identifier="BookId" version="2.0">
  <metadata xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:opf="http://www.idpf.org/2007/opf">
    <dc:title>落第騎士英雄譚 01</dc:title>
    <dc:creator opf:role="aut" opf:file-as="海空りく">海空りく</dc:creator>
    <dc:identifier id="BookId" opf:scheme="UUID">urn:uuid:5bf57cd5-b984-41a6-bed4-21c35b135bb6</dc:identifier>
    <dc:publisher>尖端出版社</dc:publisher>
    <dc:description>「只要全都聽你的就好了嘛！色鬼！」 〈魔法騎士〉，他們是一群能將靈魂轉化為魔劍，使用魔劍戰鬥的現代魔法師。黑鐵一輝就讀於培育魔法騎士的學校，但他卻極度缺！…………省略</dc:description>
    <dc:date>2017-02-14T16:00:00+00:00</dc:date>
    <dc:subject>爱情</dc:subject>
    <dc:subject>动作</dc:subject>
    <dc:subject>奇幻</dc:subject>
    <dc:subject>校园</dc:subject>
    <dc:subject>战斗</dc:subject>
    <dc:contributor opf:role="bkp">calibre (7.1.0) [https://calibre-ebook.com]</dc:contributor>
    <dc:language>zh-TW</dc:language>
    <dc:identifier opf:scheme="calibre">e3588cb5-38ed-41d3-9f5c-1873a09ddfe4</dc:identifier>
    <meta content="0.7.4" name="Sigil version"/>
    <meta name="cover" content="x000.jpg"/>
    <meta name="calibre:title_sort" content="落第騎士英雄譚 01"/>
    <meta name="calibre:series" content="落第骑士英雄谭"/>
    <meta name="calibre:series_index" content="1.0"/>
  </metadata>
  <manifest>
    <item href="Text/cover.xhtml" id="cover.xhtml" media-type="application/xhtml+xml"/>
    <item href="Text/Section0001.xhtml" id="Section0001.xhtml" media-type="application/xhtml+xml"/>
    <item href="Text/Section0002.xhtml" id="Section0002.xhtml" media-type="application/xhtml+xml"/>
 …………省略
    <item href="Text/Section0008.xhtml" id="Section0008.xhtml" media-type="application/xhtml+xml"/>
    <item href="Text/Section0009.xhtml" id="Section0009.xhtml" media-type="application/xhtml+xml"/>
    <item href="toc.ncx" id="ncx" media-type="application/x-dtbncx+xml"/>
    <item href="Images/000.jpg" id="x000.jpg" media-type="image/jpeg"/>
    <item href="Images/001.jpg" id="x001.jpg" media-type="image/jpeg"/>
    <item href="Images/002.jpg" id="x002.jpg" media-type="image/jpeg"/>
    <item href="Images/003.jpg" id="x003.jpg" media-type="image/jpeg"/>
…………省略
    <item href="Images/017.jpg" id="x017.jpg" media-type="image/jpeg"/>
    <item href="Images/018.jpg" id="x018.jpg" media-type="image/jpeg"/>
  </manifest>
  
  <spine toc="ncx">
    <itemref idref="cover.xhtml"/>
    <itemref idref="Section0001.xhtml"/>
    <itemref idref="Section0002.xhtml"/>
    <itemref idref="Section0003.xhtml"/>
    <itemref idref="Section0004.xhtml"/>
    <itemref idref="Section0005.xhtml"/>
    <itemref idref="Section0006.xhtml"/>
    <itemref idref="Section0007.xhtml"/>
    <itemref idref="Section0008.xhtml"/>
    <itemref idref="Section0009.xhtml"/>
  </spine> 这上面一堆，规定了xhtml的顺序，就是轻小说目录，也就是说新建一个xhtml文件，放在第一行即可
  
  <guide>
    <reference href="Text/cover.xhtml" title="封面" type="cover"/>
  </guide>
</package>

```

## 假设在众多.xhtml的那个文件夹，新建有 `插图.xhtml` 文件，也就是 `/OEBPS/Text/插图.xhtml`，那么/OEBPS/content.opf文件可以这样修改 :

```html
/OEBPS/content.opf文件 :

  <manifest>
    <item href="Text/插图.xhtml" id="插图测试.xhtml" media-type="application/xhtml+xml"/>  添加这一行
    <item href="Text/cover.xhtml" id="cover.xhtml" media-type="application/xhtml+xml"/>
 ……
 
  </manifest>
  

    
    
    
  <spine toc="ncx">
        <itemref idref="插图测试.xhtml"/> 添加这一行
        <itemref idref="cover.xhtml"/>
    
        ……
        
  </spine> 
```

    manifest标签的href : 
        Text/插图.xhtml : 添加自己的xhtml的路径，Text/插图.xhtml 指的是本content.opf文件的相对目录，Text文件夹下的文件
    manifest标签的id :
        插图测试.xhtml : 这个是任意名字，添加在下面   <spine toc="ncx"> 这行下的首行
        
##  构建新的 Text/插图.xhtml 前，先打开现成的xhtml看一下:   

```html
这个是 
/海空りく - 落第騎士英雄譚 01/OEBPS/Text/cover.xhtml

<?xml version="1.0" encoding="utf-8" standalone="no"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title>Cover</title>
</head>

<body>
  <div style="text-align: center; padding: 0pt; margin: 0pt;">
    <svg xmlns="http://www.w3.org/2000/svg" height="100%" preserveAspectRatio="xMidYMid meet" version="1.1" viewBox="0 0 850 1222" width="100%" xmlns:xlink="http://www.w3.org/1999/xlink">
      <image width="850" height="1222" xlink:href="../Images/000.jpg"></image>
    </svg>
  </div>
</body>
</html>
```

```html
这个是
/海空りく - 落第騎士英雄譚 01/OEBPS/Text/Section0001.xhtml

<?xml version="1.0" encoding="utf-8" standalone="no"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
  "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
  <title></title>
</head>

<body>
  <p style="text-align: center;"><img alt="000" src="../Images/000.jpg" /><br /></p>

  <h1 style="text-align: center;">&#160;落第騎士英雄譚1</h1>

  <p>「只要全都聽你的就好了嘛！色鬼！」<br /></p>

  <p>〈魔法騎士〉，他們是一群能將靈魂轉化為魔劍，使用魔劍戰鬥的現代魔法師。黑鐵一輝就讀於培育魔法騎士的學校，但他卻極度缺乏魔法才華，成績低劣，被戲稱為〈落第騎士〉。某日，異國王女史黛菈單方面對一輝提出決鬥，並且要求「敗者終生服從勝者」——一輝卻不小心擊敗身為〈A級騎士〉的史黛菈！<br /></p>

  <p>一輝的魔法才能相當差勁，但他的劍術卻極為高超！</p>

  <p>史黛菈心裡雖有千萬個不情願，她的目光卻越來越離不開一輝。</p>

  <p>而另一方面，曾經的〈落第騎士〉重新以〈無冕劍王〉的身份，在這場爭奪騎士頂點的戰爭中嶄露頭角！</p>
</body>
</html>
```  

图片html有些不一样，大概就是 :

### 以下是 /OEBPS/Text/插图.xhtml 文件:

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Image Gallery</title>
</head>
<body>

<div class="duokan-image-single illus"><img alt="图片.jpg" src="../Image/0001.jpg" /></div>
<div class="duokan-image-single illus"><img alt="图片.jpg" src="../Image/0002.jpg" /></div>
<div class="duokan-image-single illus"><img alt="图片.jpg" src="../Image/0003.jpg" /></div>
<div class="duokan-image-single illus"><img alt="图片.jpg" src="../Image/0004.jpg" /></div>
<div class="duokan-image-single illus"><img alt="图片.jpg" src="../Image/0005.jpg" /></div>

``` 
注意要duokan-image-single illus这样添加 ，才能在多看阅读app里面，进行放大查看图片

../指的是上层文件夹，比如

```
├── OEBPS
│   ├── Images
│   │    └── 000.jpg
│   ├── Text
│   │    ├─ 000.xhtml
│   │    └── 001.jpg

000.xhtml中:
<div><img alt="图片.jpg" src="../Image/000.jpg" /></div>
<div><img alt="图片.jpg" src="./001.jpg" /></div>
```

图片和xhtml文件不一定在 /海空りく01/OEBPS/Images/和 /海空りく01/OEBPS/Text/文件夹下面，也有可能直接都在一个文件夹，所以直接找到所有.jpg. jpeg后缀图片，按照相对目录，添加到xhtml文本中，再在content.opf 中添加xhtml目录即可
content.opf文件名固定，但不一定在 /OEBPS/content.opf这个路径下
