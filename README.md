# 디스코드 뮤직 봇
## 패키지 설치
```
pip install discord
pip install discord.py[voice]
pip install youtube_dl
```
## 코드
```python
import discord
from discord.ext import commands
import youtube_dl
app = commands.Bot(command_prefix='.')

ydl_opts = {  # 다운로드 옵션
    'format': 'bestaudio'
}
FFMPEG_OPTIONS = {'before_options': '-reconnect 1 -reconnect_streamed 1 -reconnect_delay_max 5',  # FFMPEG 옵션
                  'options': '-vn'}

@app.event
async def on_ready():
    await app.change_presence(status=discord.Status.online, activity=None)
    print('discord bot run')

@app.command()
async def 입장(ctx):
    print(ctx.author.voice)
    if ctx.author.voice:
      await ctx.author.voice.channel.connect()

@app.command()
async def 퇴장(ctx):
    if not app.voice_clients:
        for voice in app.voice_clients:
            if voice.guild == ctx.author.guild:
                await voice.disconnect()
                break
                
@app.command()
async def 일시정지(ctx):
    if app.voice_clients:
        for voice in app.voice_clients:
            if voice.guild == ctx.author.guild:
                if voice.is_playing(): #현재 오디오를 재생 중인지 여부
                    voice.pause()
                    await ctx.send("노래가 일시정지 되었습니다.")
                    break

@app.command()
async def 재시작(ctx):
    if app.voice_clients:
        for voice in app.voice_clients:
            if voice.guild == ctx.author.guild:
                if voice.is_paused(): #오디오를 재생하고 있지만 일시 중지됐는지 여부
                    voice.resume()
                    await ctx.send("노래가 재시작 되었습니다.")
                    break

@app.command()
async def 재생(ctx, *, msg):
    if msg.startswith('https://www.youtube.com/'): #유튜브 링크 확인
        if app.voice_clients: #봇 음성채널 존재유무 확인
            for voice in app.voice_clients: #반복문 시작
                if voice.guild == ctx.author.guild: #사용자 서버 와 봇의 서버 동일한지 확인
                    with youtube_dl.YoutubeDL(ydl_opts) as ydl:
                        info = ydl.extract_info(msg, download=False) #유튜브 영상 정보 가져오기
                        URL = info['formats'][0]['url']
                        song = discord.FFmpegPCMAudio(URL, **FFMPEG_OPTIONS) #오디오 변환
                        voice.play(song) # 노래 재생
                        await ctx.send(info['title']+" 를 재생합니다")
                    break

app.run('TOKEN')
```
## 블로그
https://camon.tistory.com/4?category=976442[here](https://camon.tistory.com/4?category=976442)
