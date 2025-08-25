#python -m pip install --upgrade pip
#pip install requests
#pip install beautifulsoup4

import csv
import requests
from bs4 import BeautifulSoup
import os

url ="https://www.imdb.com/title/tt0455275/episodes/?ref_=tt_ov_epl"
#url ="https://www.imdb.com/title/tt0903747/episodes/?ref_=tt_ov_epl"
#hdr = {'User-Agent': 'Mozilla/5.0'}
hdr = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.0.0 Safari/537.36'}
res = requests.get(url, headers=hdr)
res.raise_for_status()

soup = BeautifulSoup(res.text, "lxml")
#print(soup)

#season = soup.find_all("li", attrs={"data-testid": "tab-season-entry"})
season = soup.find_all("a", attrs={"data-testid": "tab-season-entry"})
nums_season = len(season)
text_season = []
print(len(season))
for s in season:
    print(s)
    #nums_season = s.get_text()
    text_season.append(s.get_text())
filename = "프리즌 브레이크 시즌 정보.csv"
f = open(filename, mode="w", encoding="utf-8-sig", newline="")
writer = csv.writer(f)

attributes = ["시즌", "방영일", "제목", "리뷰수", "평점", '줄거리']
writer.writerow(attributes)

img_dir_name = "프리즌 브레이크시리즈이미지"
if not os.path.isdir(img_dir_name):
      os.mkdir(img_dir_name)
cur_dir = os.getcwd()
# os.mkdir(f'{cur_dir}/{img_dir_name}')


#for season in range(1, int(nums_season)+1):
for season in text_season:
      url ="https://www.imdb.com/title/tt0455275/episodes?season={}".format(season)
      #url ="https://www.imdb.com/title/tt0903747/episodes/?season={}".format(season)
      #url =f"https://www.imdb.com/title/tt1475582/episodes?season={season}"
      res = requests.get(url, headers=hdr)
      res.raise_for_status()

      soup = BeautifulSoup(res.text, "lxml")
      #episodes = soup.find_all("article", attrs={"class": "sc-f1a948e3-1 bGxjcH episode-item-wrapper"})
      episodes = soup.find_all("article", attrs={"class": "sc-282bae8e-1 dSEzwa episode-item-wrapper"})

      for episode in episodes:
            #title = episode.find("a", attrs={"class": "sc-1318654d-8 bglHll"}).get_text()
            #date = episode.find("span", attrs={"class": "sc-1318654d-10 jEHgCG"}).get_text()
            title = episode.find("a", attrs={"class": "ipc-title-link-wrapper"}).get_text()
            if '/' in title:
              title = title.replace('/', '_')

            date = episode.find("span", attrs={"class": "sc-ccd6e31b-10 fVspdm"}).get_text()
            rating_votecount = episode.find("span", attrs={"class": "ipc-rating-star ipc-rating-star--base ipc-rating-star--imdb ratingGroup--imdb-rating"}).get_text()
            rating = rating_votecount[0:3]
            votecount = rating_votecount[8:11]
            descr = episode.find("div", attrs={"class": "ipc-html-content-inner-div"}).get_text()


            print(title, date, rating, votecount, descr)
            data_rows = [season, date, title, votecount, rating, descr]
            writer.writerow(data_rows)

            img = episode.img
            imgurl = img['src'];
            r = requests.get(imgurl, headers=hdr)
            with open(f'{cur_dir}/{img_dir_name}/{title[0:]}.jpg', "wb") as out:
                  out.write(r.content)

f.close()

list_target = [['tt1475582', '셜록'], ['tt0944947', '왕좌의 게임']]


print(f"총 시즌 수:{nums_season}")
print(f"총 시즌 제목:{text_season}")
