# Thread  
thread有優先順序,，在建立時可以指定，優先權高的執行會比優先權低的 thread也可以成為一種demon，如果有指定的話 建立thread的方式，其中一種為建立一個class，然後extend Thread class，再override run method，之後使用一般建立物件方式來使用，執行start method，而run method裡面就可以做我們要這個thread做的事情。  

另一種方式，則是implements Runnable，也是同樣要override run method，建立物件則需把該thread class丟給Thread class，由Thread instance的start method啓動我們的thread。  
