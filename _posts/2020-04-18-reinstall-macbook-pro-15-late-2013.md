---
title: "MacBook Pro 15\" late 2013重新安裝"
date: 2020-04-18 23:31:34 +0800
categories: ["生活點滴", "Python"]
---

不得不說，MacBook Pro真的比一般商用筆電還耐用，買來用了快7年還是有著不錯的效能，不過因為一直以來為了因應各式各樣的專案及學習的需求，安裝了各種不同的軟體，導致作業系統變得很凌亂，最近安裝opencv一直搞不定python3的環境衝突，只好打掉重來．

先把檔案複製到raid 0的外接硬碟上，然後準備重安裝，因為外接硬碟是用RAID輔助工具做的，一直很擔心重安裝後就不見，結果證明多慮了．

現有作業系統已經升級到Catalina，所以不需要花心思去做安裝USB，直接重新開機後按住command(⌘) + R鍵就會進入重新安裝的畫面，進入後，清除現在的卷宗再開始重新安裝，經過15~20分鐘後，就進入了設定畫面，設定後就完成了重新安裝的步驟．

進入App Store，把已經購買的APP都下載回來，XCode安裝完成後，會自帶Python 2.7.16跟3.7.3，有19.0.3的pip3，無pip，Python真的很亂，有安裝了pyenv來控制，但還不曉得是否work．

安裝Homebrew，有些工具用brew安裝，有些用安裝檔

|  |  |
| --- | --- |
| 工具 | 安裝方式 |
| Homebrew | `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"` |
| Atom | brew cask install atom |
| eclipse | 安裝檔 |
| vscode | 安裝檔 |
| drawio | brew cask install drawio |
| item2 | brew cask install iterm2 |
| docker | 安裝檔 |
| virtualbox | 安裝檔 |
| office 365 | 安裝檔 |
| chrome | 安裝檔 |
| minikube | brew install minikube |
| JDK | brew tap AdoptOpenJDK/openjdk brew cask install adoptopenjdk8 brew cask install adoptopenjdk11 |
| arduino | brew cask install arduino |

Atom跟iterm2是首次嘗試的文字編輯器跟終端機視窗，還不太習慣，新版的Docker CE自帶了kubernetes，所以安裝minikube時，會出現kubectl的link已經存在的錯誤，再下一次指令(brew link --overwrite kubernetes-cli)蓋掉Docker CE建立的link，minukube用(minikube start --driver=virtualbox )啟動．

Atom竟然不支援區塊編輯

要設定iterm2得再安裝brew install zsh-completions，裝完執行

```
rm -f ~/.zcompdump; compinit
chmod go-w '/usr/local/share'
```

然後再安裝on-my-zsh

```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

會一直提醒unsafety，執行compaudit | xargs chmod g-w,o-w

接著再去下載配色跟melos的字型

JDK安裝兩個LTS的版本，並在.zshrc中增加

```
jdk() {
        version=$1
        export JAVA_HOME=$(/usr/libexec/java_home -v"$version");
        java -version
 }
```

可以在命令列下jdk 1.8、jdk 11來切換JAVA環境，至此，經過Time Machine備份後，準備開始安裝opencv．

pyenv要在.zshrc中增加以下片段

```
PATH="$(pyenv root)/shims:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

預計要安裝opencv4，憑依著python 3.8，所以先手動安裝python 3.8.2

pyenv install 3.8.2，安裝完成後，使用pyenv global 3.8.2切換

升級pip ==> pip install --upgrade pip

但這個升級沒什麼用，近虛擬環境還是要再做一次

brew install opencv，結果還是會去裝python 3.8.2

要再安裝pip3 install opencv-python

要安裝pip3 install pylint，vscode才不會一直叫人安裝

測試opencv

```
import cv2
from cv2 import cv2 as cv

print("OpenCV version:")
print(cv.__version__)

img = cv.imread("cloud.jpeg")
gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)

cv.imshow("Over the Clouds", img)
cv.imshow("Over the Clouds - gray", gray)

cv.waitKey(0)
cv.destroyAllWindows()
```

執行起來後按任意鍵結束，這樣就算安裝成功了
