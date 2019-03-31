# Visual Studio Code中的Python單元測試
Python擴展支持unit test 使用Python的內置的單元測試框架以及pytest。雖然Nose框架本身處於維護模式，但也支持。

啟用一個測試框架之後，使用`Python: Discover Unit Tests` 命令掃描測試的項目根據當前選定的測試框架的發現模式。一旦發現，Visual Studio Code提供了各種方法來運行測試和調試測試。VS Code在Python Test Log面板中顯示單元測試輸出，包括未安裝測試框架時導致的錯誤。使用PyTest，失敗的測試也會出現在“ 問題”面板中。

## 關於單元測試的一點背景
（如果您已熟悉單元測試，則可直接跳到演練。）

一單元是特定的代碼塊進行測試，如函數或類。然後，單元測試是其他代碼段，它們專門使用各種不同的輸入來運行代碼單元，包括邊界和邊緣情況。

例如，假設您具有驗證用戶在Web表單中輸入的帳號格式的功能：
```py
def  validate_account_number_format（account_string）：
     ＃如果無效則返回false，如果有效則返回true 
    ＃ ...
```
單元測試只關注單元的接口 -its參數和返回值 - 而不是它的實現（這就是為什麼函數體中沒有顯示代碼的原因;通常你會使用其他經過良好測試的庫來幫助實現這個功能）。在此示例中，函數接受任何字符串，如果該字符串包含格式正確的帳號，則返回true，否則返回false。

要徹底測試這個函數，你想要輸入所有可能的輸入：有效字符串，錯誤輸入的字符串（關閉一個或兩個字符，或包含無效字符），字符串太短或太長，空字符串，空參數，包含控製字符的字符串（非文本代碼），包含HTML的字符串，包含注入攻擊的字符串（如SQL命令或JavaScript代碼）等。如果驗證後的字符串稍後在數據庫查詢中使用或顯示在應用程序的UI中，則測試注入攻擊等安全案例尤為重要。

然後，對於每個輸入，您可以定義函數的預期返回值（或多個值）。在此示例中，同樣，函數應僅對格式正確的字符串返回true。（數字本身是否是真實賬戶是另一種可以通過數據庫查詢在其他地方處理的問題。）

有了所有的參數和預期的返回值，你現在自己編寫測試，它只是用特定輸入調用函數的代碼片段，然後將實際返回值與預期的返回值進行比較（這種比較稱為斷言）：
```py
# Import the code to be tested
import validator

# Import the unit test framework (this is a hypothetical module)
import test_framework

# This is a generalized example, not specific to a test framework
class Test_TestAccountValidator(test_framework.TestBaseClass):
    def test_validator_valid_string():
        # The exact assertion call depends on the framework as well
        assert(validate_account_number_format("1234567890"), true)

    # ...

    def test_validator_blank_string():
        # The exact assertion call depends on the framework as well
        assert(validate_account_number_format(""), false)

    # ...

    def test_validator_sql_injection():
        # The exact assertion call depends on the framework as well
        assert(validate_account_number_format("drop database master"), false)

    # ... tests for all other cases
```

代碼的確切結構取決於您正在使用的單元測試框架，本文稍後將提供具體示例。無論如何，正如您所看到的，每個測試都非常簡單：使用參數調用函數並斷言預期的返回值。

所有測試的組合結果是您的測試報告，它告訴您功能（單元）是否在所有測試用例中按預期運行。也就是說，當一個單元通過所有測試時，您可以確信它正常運行。（測試驅動開發的實踐是您首先編寫測試的地方，然後編寫代碼以傳遞越來越多的測試，直到所有測試都通過。）

因為單元測試是一個小的，孤立的代碼片段（在單元測試中你避免了外部依賴並使用模擬數據或其他模擬輸入），它們運行起來快速而且便宜。此特性意味著您可以儘早並經常運行單元測試。開發人員通常在將代碼提交到存儲庫之前運行單元測試; 門控簽入系統也可以在合併提交之前運行單元測試。許多持續集成系統也在每次構建後運行單元測試。儘早運行單元測試通常意味著您可以快速獲得回歸，這是先前通過其所有單元測試的代碼行為的意外更改。由於測試失敗可以很容易地追溯到特定的代碼更改，因此很容易找到並解決失敗的原因，這無疑比在此過程中稍後發現問題更好！

有關單元測試的一般背景，看到單元測試維基百科。有關各種有用的單元測試示例，請參閱https://github.com/gwtw/py-sorting，這是一個包含不同排序算法測試的存儲庫。

## 示例測試演練
Python測試是Python類，它們位於與被測試代碼不同的文件中。每個測試框架都指定了測試和測試文件的結構和命名。一旦編寫測試並啟用測試框架，VS Code就會找到這些測試，並為您提供運行和調試它們的各種命令。

對於此部分，創建一個文件夾並在VS Code中打開它。然後創建一個名為inc_dec.py以下代碼的文件進行測試：

```py
def increment(x):
    return x + 1

def decrement(x):
    return x - 1
```
使用此代碼，您可以體驗在VS代碼中使用測試，如以下部分所述。

## 啟用測試框架
默認情況下禁用Python中的單元測試。為了使單元測試，設置一個且只有一個以下設置為true： `python.unitTest.unittestEnabled, python.unitTest.pyTestEnabled, and python.unitTest.nosetestsEnabled.`。每個框架還具有特定的配置設置，如測試配置設置中所述。

一次只啟用一個測試框架非常重要。因此，當您啟用一個框架時，請務必禁用其他框架。

啟用測試框架時，如果框架包尚未存在於當前激活的環境中，則VS Code會提示您安裝該框架包：

VS Code提示在啟用時安裝測試框架

設置以啟用unittest
“ python.unitTest.unittestEnabled ”： true，
 “ python.unitTest.pyTestEnabled ”： false，
 “ python.unitTest.nosetestsEnabled ”： false，
設置以啟用pytest
“ python.unitTest.unittestEnabled ”： false，
 “ python.unitTest.pyTestEnabled ”： true，
 “ python.unitTest.nosetestsEnabled ”： false，
設置以啟用Nose
“ python.unitTest.unittestEnabled ”： false，
 “ python.unitTest.pyTestEnabled ”： false，
 “ python.unitTest.nosetestsEnabled ”： true，
## 創建測試
每個測試框架都有自己的約定，用於命名測試文件和在其中構建測試，如以下部分所述。每種情況都包括兩種測試方法，其中一種是為了演示而故意設置為失敗。

由於Nose處於維護模式而不建議用於新項目，因此下面的部分中僅顯示了unittest和pytest示例。（Nose2，Nose的繼承者，只是插件的單元測試，所以它遵循這裡顯示的單元測試模式。）

### 單元測試的測試
`test_unittest.py`使用兩種測試方法創建一個包含測試類的文件：
```py
import inc_dec    # The code to test
import unittest   # The test framework

class Test_TestIncrementDecrement(unittest.TestCase):
    def test_increment(self):
        self.assertEqual(inc_dec.increment(3), 4)

    def test_decrement(self):
        self.assertEqual(inc_dec.decrement(3), 4)

if __name__ == '__main__':
    unittest.main()
```

### 用pytest 測試
創建一個名為test_pytest.py包含兩個測試方法的文件：
```py
import inc_dec    # The code to test

def test_increment():
    assert inc_dec.increment(3) == 4

def test_decrement():
    assert inc_dec.decrement(3) == 4
```

## 測試發現
VS Code使用當前啟用的單元測試框架來發現測試。您可以使用`Python：Discover Unit Tests` 命令隨時觸發測試發現。

`python.unitTest.autoTestDiscoverOnSaveEnabled`默認設置為`true`，表示每次保存測試文件時都會自動執行測試發現。要禁用此功能，請將值設置為false。

測試發現應用當前框架的發現模式（可以使用“ 測試”配置設置進行自定義）。默認行為如下：

python.unitTest.unittestArgs：.py在頂級項目文件夾的名稱中查找任何帶有“test”的Python（）文件。所有測試文件都必須是可導入的模塊或包。您可以使用-p配置設置自定義文件匹配模式，並使用該-t設置自定義文件夾。

python.unitTest.pyTestArgs：查找.py名稱以“test_”開頭或以“_test”結尾的任何Python（）文件，位於當前文件夾和所有子文件夾中的任何位置。

提示：有時不會發現放置在子文件夾中的單元測試，因為無法導入此類測試文件。要使它們可導入，請創建__init__.py在該文件夾中命名的空文件。

如果發現成功，狀態欄將顯示“運行測試”：

狀態欄顯示成功的測試發現失敗

如果發現失敗（例如，未安裝測試框架），則會在狀態欄上看到通知。選擇通知可提供更多信息：

狀態欄顯示測試發現失敗

一旦VS碼識別測試中，它提供了多種方法中的說明來運行這些測試運行測試。最明顯的方法是直接出現在編輯器中的CodeLens裝飾，允許您輕鬆運行單個測試方法，或者使用unittest測試類：

測試出現在VS Code編輯器中的單元測試代碼的裝飾品

測試出現在VS代碼編輯器中的pytest代碼的裝飾句

注意：目前，Python擴展程序不提供打開或關閉裝飾的設置。要建議不同的行為，請在vscode-python存儲庫上提交問題。

對於Python，測試發現還會在VS Code活動欄上使用圖標激活測試資源管理器。該測試資源管理器可以幫助您顯現，導航和運行單元測試：

用於Python單元測試的VS Code Test Explorer

運行測試
您可以使用以下任何操作運行測試：

打開測試文件，選擇顯示在測試方法或類上方的Run Test CodeLens裝飾，如上一節所示。此命令僅運行一個方法或僅運行類中的那些測試。

選擇狀態欄上的運行測試（可根據結果更改外觀），

VS Code狀態欄上的測試命令

然後選擇其中一個命令，如運行所有單元測試或運行失敗的單元測試：

使用“運行測試”狀態欄命令後顯示的測試命令

在Test Explorer中：

要運行所有已發現的測試，請選擇Test Explorer頂部的播放按鈕：

通過Test Explorer運行所有測試

要運行特定的測試組或單個測試，請選擇文件，類或測試，然後選擇該項右側的播放按鈕：

通過Test Explorer在特定範圍內運行測試

在資源管理器中右鍵單擊文件，然後選擇“運行所有單元測試”，該測試將在該文件中運行測試。

從命令選項板中，選擇任何運行單元測試命令：

命令選項板上的Python單元測試命令

命令	描述
調試所有單元測試	請參閱調試測試。
調試單元測試方法	請參閱調試測試。
運行所有單元測試	在工作空間及其子文件夾中搜索並運行所有單元測試。
運行當前單元測試文件	在當前在編輯器中查看的文件中運行測試。
運行失敗的單元測試	重新運行先前測試運行中失敗的任何測試。如果尚未運行任何測試，則運行所有測試。
運行單元測試文件	提示輸入特定的測試文件名，然後在該文件中運行測試。
運行單元測試方法	提示要運行的測試名稱，為測試名稱提供自動完成功能。
顯示單元測試輸出	打開Python Test Log面板，其中包含有關傳遞和失敗測試以及錯誤和跳過測試的信息。
測試運行後，VS Code會直接在編輯器和測試資源管理器中使用CodeLens裝飾顯示結果。結果顯示為單個測試以及包含這些測試的任何類和文件。失敗的測試也在編輯器中用紅色下劃線裝飾。

在單元測試類和測試資源管理器中測試結果

VS Code還在Python Test Log輸出面板中顯示測試結果（使用View > Output菜單命令顯示Output面板，然後從右側的下拉列表中選擇Python Test Log）：

在Python測試日誌輸出面板中測試結果

使用PyTest，失敗的測試也會出現在“ 問題”面板中，您可以在其中雙擊某個問題以直接導航到測試：

在“問題”面板中測試pytest的結果

調試測試
因為單元測試本身就是源代碼，所以它們很容易出現代碼缺陷，就像它們測試的生產代碼一樣。因此，您可能偶爾需要在調試器中單步執行並分析單元測試。

例如，test_decrement之前給出的函數失敗，因為斷言本身是錯誤的。以下步驟演示瞭如何分析測試：

在test_decrement函數的第一行設置斷點。

選擇該函數上方的Debug Test裝飾或Test Explorer中該測試的“bug”圖標。VS Code啟動調試器並在斷點處暫停。

在“ 調試控制台”面板中，輸入inc_dec.decrement(3)以查看實際結果是2，而測試中指定的預期結果是不正確的值4。

停止調試器並更正錯誤代碼：

＃單元測試
自 .assertEqual（inc_dec.decrement（ 3）， 2）

＃ pytest 
斷言 inc_dec.decrement（ 3） ==  4
保存文件並再次運行測試以確認它們通過，並看到CodeLens裝飾也指示傳遞狀態。

注意：運行或調試單元測試不會自動保存測試文件。始終確保在運行之前保存對測試的更改，否則您可能會對結果感到困惑，因為它們仍然反映了文件的先前版本！

在Python的：調試所有測試和Python的：調試單元測試方法的命令（在命令面板和狀態欄菜單兩者）分別推出針對所有測試和一個測試方法調試。您還可以使用測試資源管理器中的“錯誤”圖標為所選範圍內的所有測試以及所有已發現的測試啟動調試器。

調試器對於單元測試的工作方式與其他Python代碼相同，包括斷點，變量檢查等。有關更多信息，請參閱Python調試配置和常規VS Code Debugging文章。

測試配置設置
使用Python進行單元測試的行為是由一般設置和設置驅動的，這些設置和設置特定於您啟用的任何框架。

常規設置
設置
（python.unitTest。）	默認	描述
autoTestDiscoverOnSaveEnabled	true	指定在保存單元測試文件時是啟用還是禁用自動運行測試發現。
CWD	空值	指定單元測試的可選工作目錄。
DEBUGPORT	3000	用於調試UnitTest測試的端口號。
outputWindow	"Python Test Log"	用於單元測試輸出的窗口。
promptToConfigure	true	指定VS代碼是否在發現潛在測試時提示配置測試框架。
Unittest配置設置
設置
（python.unitTest。）	默認	描述
unittestEnabled	false	指定是否啟用UnitTest作為測試框架。應禁用所有其他框架。
unittestArgs	["-v", "-s", ".", "-p", "*test*.py"]	傳遞給unittest的參數，其中由空格分隔的每個元素是列表中的單獨項。有關默認值的說明，請參見下文。
UnitTest的默認參數如下：

-v設置默認詳細程度。刪除此參數以獲得更簡單的輸出。
-s .指定用於發現測試的起始目錄。如果在“test”文件夾中有測試，請將參數更改為-s test（"-s", "test"在arguments數組中）。
-p *test*.py是用於查找測試的發現模式。在這種情況下，它.py是包含單詞“test” 的任何文件。如果以不同方式命名測試文件，例如在每個文件名後附加“_test”，則使用類似於*_test.py數組的相應參數的模式。
要在第一次失敗時停止測試運行，請將fail fast選項添加"-f"到arguments數組中。

有關完整的可用選項集，請參閱unittest命令行界面。

Pytest配置設置
設置
（python.unitTest。）	默認	描述
pyTestEnabled	false	指定是否啟用PyTest作為測試框架。應禁用所有其他框架。
pyTestPath	"pytest"	PyTest的路徑。如果PyTest位於當前環境之外，請使用完整路徑。
pyTestArgs	[]	傳遞給PyTest的參數，其中由空格分隔的每個元素都是列表中的單獨項。請參閱PyTest命令行選項。
您還可以使用PyTest Configuration中pytest.ini所述的文件配置pytest 。

注意 如果安裝了pytest-cov coverage模塊，則VS Code在調試時不會在斷點處停止，因為pytest-cov使用相同的技術來訪問正在運行的源代碼。要防止此行為，請--no-cov在pyTestArgs調試測試時包括。（有關更多信息，請參閱pytest-cov文檔中的Debuggers和PyCharm。）

鼻子配置設置
設置
（python.unitTest。）	默認	描述
nosetestsEnabled	false	指定是否啟用Nose作為測試框架。應禁用所有其他框架。
nosetestPath	"nosetests"	鼻子的路徑。如果Nose位於當前環境之外，請使用完整路徑。
nosetestArgs	[]	傳遞給Nose的參數，其中由空格分隔的每個元素是列表中的單獨項。請參閱鼻子使用選項。
您還可以配置的鼻子用.noserc或nose.cfg作為描述文件前端構造。

也可以看看
Python環境 - 控制使用哪個Python解釋器進行編輯和調試。
設置參考 - 探索VS Code中與Python相關的所有設置。