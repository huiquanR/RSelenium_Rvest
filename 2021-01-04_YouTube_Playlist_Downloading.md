# Download a YouTube Playlist as a Table with R!

```R
#### 2020.01.04 Read HTML ####

# Clean Envir 
rm(list = setdiff(ls(), c("Read_YouTube_Playlist")))

Read_YouTube_Playlist = function(WEB_LINK) {
  # load packages necessary #
  library(RSelenium)
  library(rvest)
  library(stringr)
  library(XML)
  library(xml2)
  library(htmltools)
  
  # initiate webDriver #
  RD = RSelenium::rsDriver(port=as.integer(sample(1:9999, 
                                                  1, 
                                                  replace=F)), # generate random int each time
                           browser = c("chrome"), 
                           chromever = "87.0.4280.88")
  
  # define driver #
  remDr = RD[["client"]]
  
  # navigate to page #
  remDr$navigate(WEB_LINK)
  
  # wait for page to load #
  Sys.sleep(3.0)
  
  # please note that findElements & findElement are different functions #
  numElem = remDr$findElement(using = 'css selector', 
                              value = '#stats > yt-formatted-string:nth-child(1) > span:nth-child(1)')
  a = numElem$getElementText()
  a = as.numeric(unlist(a))
  b = round(a * 0.01) + 1
  
  # Scroll down to the end of page #
  RunToEnd <- function(b) {
    webElem <- remDr$findElement("css", "body")
    i <- 1 
    while (i < b) { 
      webElem$sendKeysToElement(list(key = "end"))
      i = i + 1
      Sys.sleep(4.0) # sleep for seconds to wait until page loaded
    }
  }
  RunToEnd(b)
  
  # analyze HTML page contents #
  html = read_html(remDr$getPageSource()[[1]]) # notice that we're getting [[1]] from a list
  web = html
  title = as.character(web %>% html_nodes("title") %>% html_text())
  title
  t1 = web %>% 
    html_nodes(xpath = '//*[@id="contents"]/ytd-playlist-video-renderer') %>% 
    html_text()
  t2 = web %>% 
    html_nodes(xpath = '//*[@id="content"]/a') %>% 
    html_attr("href")
  t3 = paste('https://www.youtube.com/', t2, sep ="")
  t4 = substr(t3, 1, stringr::str_locate(t3, 
                                         pattern = "&list=")[1] - 1)
  t5 = cbind(t1, t4)
  DF = data.frame(t5)
  Sys.sleep(0.056)
  assign(title, DF, envir = .GlobalEnv)
}


##
link = "https://www.youtube.com/playlist?list=PL7fSzNardl1Y1PiiNdb2YhWs9OfWWyfY9"
a = Read_YouTube_Playlist(link)

library(xlsx)
xlsx::write.xlsx(a, "d:/TONY-ZHANG-SONGS-YOUTUBE.xlsx")
```
