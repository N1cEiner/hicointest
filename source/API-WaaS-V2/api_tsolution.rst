III. Docking scheme
====================

In order to enable customers to access WAAS services more conveniently and quickly, the platform has sorted out a set of relatively perfect technology docking schemes based on the docking experience of previous customers. The details are as follows:

The whole program is divided into four processes:

1.Obtain the deposit address

2.User deposit

3.User withdrawl

4.Financial reconciliation

Note: If the client's project time is urgent, the necessary processes [obtaining the deposit address], [user charging the deposit], [user withdrawing the deposit], and [financial reconciliation] can be developed in the second stage.

1.Obtain the deposit address
-------------------

1）Customer registers with WAAS system (email address is recommended)

2）After successful registration, the address can be obtained through UID of WAAS system

Note: Getting the address in advance is conducive to efficient address allocation after user registration; If the user registers and then obtains the address through the interface, the acquisition may fail because of other reasons such as the network, resulting in the user can not normally complete the business.

.. image:: images/api_tsoulution_zhunbei.png
   :width: 400px
   :align: center

2.User deposit
-------------------

1）The user completes registration in the customer's system;

2）When the user checks the currency recharge address at the front end, the customer system will assign the currency address to the user;

3）Users charge coins to the address;

4）The WAAS system monitors the transaction of coins in the block chain address, accounts for the customer in the WAAS system, and proactively notifies the customer system or proactively obtains it by the customer system;

5）After the customer system confirms the validity of the coin charging information, it will be posted to the user's account;

6）The customer system notifies the user of successful coin charging.


.. image:: images/api_tsoulution_chongzhi.png
   :width: 600px
   :align: center


Note: The asynchronous callback of WAAS system will be triggered when each order is final, and it can be sent up to 5 times per day;

Timed task: a total of five callbacks

Notification time: 1s for the first time, 2min for the second time, 8min for the third time, 32min for the fourth time, 128min for the fifth time

Callback logic：

If the callback is successful, update the callback status;

If the callback fails, continue the callback and update the interval between the next callback;

When the callback fails five times, the callback is stopped


3.User withdrawl
-------------------

1）The user initiates the withdrawal in the customer system;

2）Inform the WAAS system after the customer system has been approved;

3）The WAAS system shall confirm the withdrawal information to the customer system twice;

4）After the customer system confirms that the withdrawal information is valid, the WAAS system verifies the withdrawal information, and then the payment is initiated;

5）The WAAS system monitors the status of currency withdrawal orders, and actively notifies the customer system of the completion of currency withdrawal or actively acquires it by the customer system;

6）The customer system informs the user that the withdrawal is successful.


.. image:: images/api_tsoulution_tibi.png
   :width: 600px
   :align: center




4.Financial reconciliation
-------------------

1）Account checking between the customer system and WAAS system on a periodic basis (daily is recommended)

2）At 0 o 'clock the next day, get all the deposits and withdrawals of the previous day as well as the consumed orders of collecting miners' fees

3）The customer system orders are compared with those in the WAAS system

4）If the order quantity, amount and status are correct, the reconciliation will be successful; Otherwise, if the reconciliation is abnormal, contact WAAS technician to help deal with it


.. image:: images/api_tsoulution_duizhang.png
   :width: 400px
   :align: center


Note: In the WAAS system, there are three kinds of cost expenses in the tripartite system: collecting miners' fee, extracting miners' fee, and profit-sharing commission;

a) Collecting Miner Fee: With the currency of account type, after charging the currency, the funds on the address will be collected to the hot wallet address and collected to the block chain network. Part of the main chain block chain transaction will consume the miner fee, and the cost of which will be borne by the customer. This part of capital expenditure needs to contact us to deal with;

b) Miner's fee: Miner's fee (in some currencies) shall be borne by the three parties when withdrawing coins to the address of non-WAAS alliance and using the block chain network. This part of capital expenditure can be viewed directly in the coin withdrawal order;

c) Distribution fee: temporarily not charged.
