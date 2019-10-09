# 觀念
### Access Right Modifier
interface不能用final  
enum不能用abstract class只能用public, abstract, final, default來修飾  
protected: 在不同的package, 需要繼承才能存取  
default: 只要在同一個package, 就可以存取  
***

### Interface
interface裡面宣告的方法隱含public修飾，所以即便不加，也是代表public，因此, 在實作此interface的類別，該方法也必需主動加上public修飾。  
interface也可以含有資料成員，不過這些成員都會自動被加入final static修飾。  
***

### OCA
* 物件可以是實體或是抽象的，如「提款機」是實體的，「付款」則比較抽象。  
* 物件導向分析思考  
    * 訪談，記錄需求項目；同時建立user case flow。  
    * 開始分類，定義有哪些類別。  
    * 把類別所需的屬性及行為定義出來，同時若屬性有關聯到其他類別則在該屬性名稱前加上星號(*)。  
    * 用UML來塑模類別之間的關係。  
* 程式執行時，每個產生的物件都會在記憶體裡佔據一塊空間，具有獨立記憶體位址，稱為「實例(instance)」。  
* 方法宣告的修飾詞如public、static是形容詞，其順序先後不影響方法的意義。  
* final用在class，表示此class無法被繼承(inhertance)；用在方法，表示無法被覆寫(override)；用在屬性，表示無法被更改值。  
* 實例變數（又稱類別屬性或成員），若在使用前沒有給初值，則java會自行給預設值。  
* 區域變數（方法內的），若在使用前沒有給初值，則無法通過編譯。  
* 宣告型別（reference type）」與所參照的「物件型態（object type）」實際上並沒有一定要相同的限制。  
* JVM記憶體分類  
    * Global  
        * static variable  
    * Stack  
        * Primitive type variable  
        * Primitive value  
        * Reference type variable  
    * Heap  
        * Refrence type instance  
* 記住，String字串是immutable，所以只要對String字串做變動，就會在String Pool產生新的字串物件；而StringBuffer and StringBuilder則不會，因為是mutable。  
* 陣列「array」是一種「容器物件(container object)」，可以裝載「多個」且「單一型態」的「基本型別/參考型別」。  
* 基本上，基本型別/參考型別宣告的變數，都是放在stack記憶體裡。而基本型別的值也是放在stack，參考型別產生的物件實體(instance)則放在heap。  
* ArrayList類別只存放參考型別的物件，不接受基本型別； 但可以改用基本型別的包覆類別。  
* 用static修飾的的方法稱為類別方法，而變數則稱為類別變數，在記憶體中只會有一份。  
* 類別方法內的使用的變數只能是該方法所傳入的參數或是方法內宣告的變數及類別變數/方法。  
***

### JAVA字串剖析
在Java中為了效率考量，以""包括的字串，只要內容相同（序㓚、大小寫相同），無論在程式碼中出現幾次，JVM都只會建立一個String實例，並在字串池（String pool）中維護。 PS. 前提是不能使用到new關鍵字，一但用了new，即便內容一樣，但是就是有另一個獨立的實例在記憶體中。  

所以要比較字串實際字元內容是否相同時，要使用equals()，記住==用在物件就只是比較是否參考到同一物件。  

用+來連接字串會產生新的實例，所以避免在重覆性的迴圈中做字串串接；可使用StringBuilder或StringBuffer來做。 StringBuilder用在單機非多執行緒情況下較佳，因為它不處理同步問題。 StringBuffer會處理同步問題，適用在多執行緒下。  
***

### 基礎語法