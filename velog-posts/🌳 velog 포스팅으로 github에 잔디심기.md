<span style="background-color: skyblue; font-size: 1.2rem; font-weight: 800; color: black;">
  😭 폴더 생성과 파일생성까지는 어렵지 않아요!! 그런데 정말 많은 시간을 소비했던 Permission 에러... 어떻게 해결했을까요!!
  </span>


<blockquote>
<p>📎 Reference 
✔ [velog와 github 연동 : 벨로그 글쓰고 잔디심기 🌱] (<a href="https://velog.io/@sooozi/velog%EC%99%80-github-%EC%97%B0%EB%8F%99-%EB%B2%A8%EB%A1%9C%EA%B7%B8-%EA%B8%80%EC%93%B0%EA%B3%A0-%EC%9E%94%EB%94%94%EC%8B%AC%EA%B8%B0">https://velog.io/@sooozi/velog%EC%99%80-github-%EC%97%B0%EB%8F%99-%EB%B2%A8%EB%A1%9C%EA%B7%B8-%EA%B8%80%EC%93%B0%EA%B3%A0-%EC%9E%94%EB%94%94%EC%8B%AC%EA%B8%B0</a>)
✔ <a href="https://velog.io/@rimgosu/velog-%EA%B8%80-%EC%9E%91%EC%84%B1-%EC%8B%9C-%EC%9E%90%EB%8F%99%EC%9C%BC%EB%A1%9C-github%EC%97%90-%EC%BB%A4%EB%B0%8B%ED%95%98%EA%B8%B0">velog 글 작성 시, 자동으로 github에 커밋하기</a>
✔ [Permission denied to github-actions[bot]] (<a href="https://stackoverflow.com/questions/72851548/permission-denied-to-github-actionsbot">https://stackoverflow.com/questions/72851548/permission-denied-to-github-actionsbot</a>)</p>
</blockquote>
<h2 id="1-github에-새-레포지토리-만들기">1. github에 새 레포지토리 만들기</h2>
<p><strong>📌 <code>public</code>으로 생성</strong>
<img alt="" src="https://velog.velcdn.com/images/tomato_o/post/7c44ce24-207e-4c4d-8f0b-224ffd17e8a8/image.png" /></p>
<h2 id="2-레포지토리에-폴더-및-파일-생성">2. 레포지토리에 폴더 및 파일 생성</h2>
<p><strong>📌<code>.github</code>폴더 안에 <code>workflows/update_blog.yml</code> 생성</strong></p>
<pre><code>velog_hub
├─ .github
|   └─ workflows
|      └─ update_blog.yml
└─ scripts
   └─ update_blog.py</code></pre><p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/706c5415-09c6-4391-ac58-9c54d574ac1d/image.png" /></p>
<h2 id="3-github-action">3. github action</h2>
<h3 id="3-1-update_blogyml-파일-생성">3-1. <code>update_blog.yml</code> 파일 생성</h3>
<p>  *<em>📌 <code>permissions</code> 꼭 작성해주기!! <a href="https://api.velog.io/rss/@tomato_o#6-%ED%95%B4%EA%B2%B0">인증문제 해결법 바로가기&gt;&gt;&gt;</a> *</em></p>
<blockquote>
<p>매일 자정 / push 될 때 파이썬 스크립트실행</p>
</blockquote>
<pre><code>name: Update Blog Posts

on:
  push:
    branches:
      - main # 또는 워크플로우를 트리거하고 싶은 브랜치 이름
  schedule:
    - cron: &quot;0 0 * * *&quot; # 매일 자정에 실행

jobs:
  update_blog:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Push changes
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'
          git push https://&lt;username&gt;${{ secrets.GH_PAT }}@github.com/&lt;username&gt;/velog_hub.git

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: &quot;3.x&quot;

      - name: Install dependencies
        run: |
          pip install feedparser gitpython

      - name: Run script
        run: python scripts/update_blog.py

    # 사용자 인증
    permissions:
      contents: write</code></pre><h3 id="3-2-update_blogpy-파일-생성">3-2. <code>update_blog.py</code> 파일 생성</h3>
<pre><code>import feedparser
import git
import os

# 벨로그 RSS 피드 URL
# example : rss_url = 'https://api.velog.io/rss/@soozi'
rss_url = 'https://api.velog.io/rss/@&lt;velog_아이디&gt;'

# 깃허브 레포지토리 경로
repo_path = '.'

# 'velog-posts' 폴더 경로
posts_dir = os.path.join(repo_path, 'velog-posts')

# 'velog-posts' 폴더가 없다면 생성
if not os.path.exists(posts_dir):
    os.makedirs(posts_dir)

# 레포지토리 로드
repo = git.Repo(repo_path)

# RSS 피드 파싱
feed = feedparser.parse(rss_url)

# 각 글을 파일로 저장하고 커밋
for entry in feed.entries:
    # 파일 이름에서 유효하지 않은 문자 제거 또는 대체
    file_name = entry.title
    file_name = file_name.replace('/', '-')  # 슬래시를 대시로 대체
    file_name = file_name.replace('\\', '-')  # 백슬래시를 대시로 대체
    # 필요에 따라 추가 문자 대체
    file_name += '.md'
    file_path = os.path.join(posts_dir, file_name)

    # 파일이 이미 존재하지 않으면 생성
    if not os.path.exists(file_path):
        with open(file_path, 'w', encoding='utf-8') as file:
            file.write(entry.description)  # 글 내용을 파일에 작성

        # 깃허브 커밋
        repo.git.add(file_path)
        repo.git.commit('-m', f':sparkles: new post: {entry.title}')

# 변경 사항을 깃허브에 푸시
repo.git.push()</code></pre><h2 id="4-pat-권한-받기">4. PAT 권한 받기</h2>
<h3 id="4-1-token-생성">4-1. token 생성</h3>
<p><strong>📌 받은 <code>token</code> 복사 (딱 한 번만 볼 수 있으니 반드시 복사!!!)</strong></p>
<blockquote>
<p><code>github 계정</code> -&gt; <code>Settings</code> -&gt; <code>Developer Settings</code> 
 -&gt; <code>Personal access tokens (classic)</code> -&gt; <code>Generate New Token</code> 
 -&gt; <code>Note</code> 작성 -&gt; <code>repo, workflow</code> 클릭 -&gt; <code>Generate new token</code>
 <img alt="" src="https://velog.velcdn.com/images/tomato_o/post/949c02a0-d441-4ae7-9340-72631f4c49eb/image.png" /></p>
</blockquote>
<h3 id="4-2-레포지토리velog_hub에-secret-설정">4-2. 레포지토리(velog_hub)에 secret 설정</h3>
<blockquote>
<p><code>velog_hub</code> -&gt; <code>Settings</code> -&gt; <code>Actions secrets and variables</code> -&gt; <code>Actions</code> -&gt; <code>New Repository Secret</code>
✔ Name : GH_PAT
✔ Secret : [2번에서 발급받은 토큰]
<img alt="" src="https://velog.velcdn.com/images/tomato_o/post/2d9933a9-4c76-4895-bfa2-cd1a804b4bce/image.png" /></p>
</blockquote>
<h2 id="5-여기까지-push">5. 여기까지 push!</h2>
<span style="background-color: pink; font-size: 1.5rem; font-weight: 800;">
  😵 그리고 장렬하게 실패
</span>

<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/c0a75194-ad44-49ae-9635-68b79eda64a3/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/64046d4b-7fc0-4321-93a3-60aa37544f0a/image.png" /></p>
<h2 id="6-해결">6. 해결</h2>
<h3 id="6-1-원격-저장소-url-변경">6-1. 원격 저장소 URL 변경</h3>
<p><strong>📌 권한 인증필요 - <code>원격 저장소 URL</code>을 올바르게 변경해야함!!</strong> </p>
<pre><code>$ git remote set-url origin https://
&quot;user-name&quot;@github.com/&quot;user-name&quot;/&quot;repository-name&quot;.git</code></pre><p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/8fb89f64-677b-43e3-a569-ed882e6ba9d6/image.png" /></p>
<h3 id="6-2-update_blogyml-파일에서-github-주소-수정">6-2. <code>update_blog.yml</code> 파일에서 github 주소 수정</h3>
<pre><code>run: |
   git config --global user.name 'github-actions[bot]'
   git config --global user.email 'github-actions[bot]@users.noreply.github.com'
   git push https://&lt;USERNAME&gt;${{ secrets.GH_PAT }}@github.com/&lt;USERNAME&gt;/velog_hub.git</code></pre><span style="background-color: pink; font-size: 1.5rem; font-weight: 800;">
🤢 6-2, 6-3을 시행한 결과 -> 응 또 실패야~ 또 Permission 에러
</span>

<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/d6f2a7c7-7896-4429-ad3e-bd93ae694ed0/image.png" />
(fix: 사용자 인증 커밋은 그냥 주석만 추가한 것)</p>
<h3 id="6-3-에러-메시지에-주목하라">6-3. 에러 메시지에 주목하라!</h3>
<pre><code>remote: Permission to USERNAME/velog_hub.git denied to github-actions[bot]</code></pre><blockquote>
<p><strong>📌 <code>github-actions[bot]</code>이 <code>USERNAME/velog_hub.git</code>에 접근할 권한이 없다!!</strong>
<strong>=&gt; 그렇다면 <code>github-actions[bot]</code>이 접근할 권한을 설정해줘야겠네?</strong></p>
</blockquote>
<h3 id="✨-해결방법-1">✨ 해결방법 1</h3>
<p>📌 Settings -&gt; Actions -&gt; General 에서 <code>Workflow Permissions</code> 설정하기 </p>
<span style="background-color: red; font-size: 1.5rem; font-weight: 800;">
😱 그러나 이 방법은 보안상 문제가 있음...
</span>


<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/967bc8e2-1b18-4857-80dd-1d50d919d31a/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/b643b53c-4bfe-4fd2-8049-40df91898cfb/image.png" /></p>
<h3 id="✨-해결방법-2---채택">✨ 해결방법 2 -&gt; 채택!!!</h3>
<p>📌 <code>update_blog.yml</code> 파일에 <code>permissions</code> 속성 추가하기</p>
<pre><code>jobs:
  update_blog:
      ...
    # 아래 내용 추가!!!
    permissions:
      contents: write</code></pre><blockquote>
<p><code>contents: write</code> : 워크플로우가 리포지토리의 콘텐츠를 수정하고 커밋/푸시할 수 있는 권한을 가지게 됨!!</p>
</blockquote>
<h3 id="💣-차이점">💣 차이점</h3>
<table>
<thead>
<tr>
<th align="left"><center> <code>Workflow Permissions</code> 설정 </center></th>
<th><code>permissions</code> 속성 추가</th>
</tr>
</thead>
<tbody><tr>
<td align="left">레포지토리의 <strong>모든 워크플로우</strong>에 대해 일괄적으로 적용</td>
<td>워크플로우 파일 내에서 각 워크플로우에 대해 필요한 최소한의 <strong>권한을 개별적</strong>으로 설정. 따라서 필요하지 않은 권한을 부여하지 않음으로써 <strong>보안 강화</strong></td>
</tr>
</tbody></table>
<h2 id="7-성공">7. 성공!!!!</h2>
<p><img alt="" src="https://velog.velcdn.com/images/tomato_o/post/c9c5b76b-65c3-4374-a045-53d8757aacd6/image.png" /></p>
<h2 id="8-후기">8. 후기</h2>
<p>다음포스팅에 계속...</p>