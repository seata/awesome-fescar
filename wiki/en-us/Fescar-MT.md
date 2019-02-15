[^title]: MT mode
[^author]: kmmshmily
[^Date]: 2019-02-13

# Manual Transaction Mode

Review the description in the overview: a distributed global transaction, the whole is a model of **the two-phase commit**. A global transaction consists of several branch transactions that meet the model requirements of **the two-phase commit**, which requires each branch transaction to have its own:

- One phase prepare behavior
- Two phase commit or rollback behavior

![Overview of a global transaction](https://upload-images.jianshu.io/upload_images/4420767-e48f0284a037d1df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

According to the two phase behavior pattern，We divide the branch transaction into **Automatic (Branch) Transaction Mode** and **Manual (Branch) Transaction Mode**.

The AT mode（[Reference Link TBD]()）is based on the **Relational Database** that **supports local ACID transactions**：

- One phase prepare behavior: In the local transaction, the business data update and the corresponding rollback log record are submitted together.
- Two phase commit behavior: Immediately ended successfully, **Auto** asynchronous batch cleanup of the rollback log.
- Two phase rollback behavior: By rolling back the log, **automatic** generates a compensation operation to complete the data rollback.

Accordingly, the MT mode does not rely on transaction support for the underlying data resources:

- One phase prepare behavior: Call the prepare logic of **custom** .
- Two phase commit behavior:Call the commit logic of **custom** .
- Two phase rollback behavior:Call the rollback logic of **custom** .

The so-called MT mode refers to the support of the branch transaction of **custom** into the management of global transactions.

