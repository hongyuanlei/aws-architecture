## RPO and RTO: Understanding the Differences

Disasters come in many forms. Corruption, theft, loss, or natural disaster can all take down your applications and destroy your data. In an ideal world, your data protection infrastructure would immediately restore all applications and data right at the time and point of failure.

But this is the real world. It is possible to immediately failover an application and to continuously replicate its data for near-zero loss. But these operations are resource-consuming and expensive. Realistically IT needs to set different recovery time and point objectives according to their buget, resources and application priority.

We call these two objectives Recovery Time Objective(RTO) and Recovery Point Objective(RPO). They are realted, and both are necessary to application and data recovery. They are also different metrics with different purposes.

### Recovery Time Objective(RTO)

**RTO refers to how much time an application can be down without causing significant damage to the business**. Some applications can be down for days without significant consequences. Some high priority applications can only be down for a few seconds without incurring employee irritation, customer anger and lost business.

RTO is not simply the duration of time between loss and recovery. The objective also accounts for the steps IT must take to restore the application and its data. if IT has invested in failover services for high priority applications, then they can safely express RTO in seconds.(IT must still restore the on-premises environment. But since the application is processing in the cloud, IT can take the time it needs)

**Your RTO mission is to categorize applications by priority and potential business loss and match your resources accordingly**. For example, typical plans for near-zero RTOs will require failover services. 4-hour RTOs allow for on-premises recovery starting with bare metal restore and ending with full application and data availability. For 8+ hour RTOs, IT may sign maintenance contracts with local system integrators.

### Recovery Point Objective(RPO)

**Recovery Point objectives refer to your company's loss tolerance: the amount of data that can be lost before significant harm to the business occurs**. The objective is expressed as a time measurement from the loss event to the most recent preceding backup.

If you back up all or most of your data in regularly scheduled 24-hour increments, then in the worst-case scenario you will lost 24 hours' worth of data. For some applications this is acceptable. For others it is absolutely not.

For example, if you have a 4-hour RPO for an application then you will have a maximum 4-hour gap between backup and data loss. Having a 4-hour RPO does not necessarily mean you will lose 4 hours' worth of data. Should a word procssing a application go down at midnight and come up by 1:15 am, you might not have much(or any) data to lose. But if a busy application goes down at 10am and isn't restored until 2:00pm, you will potentially lose 4 hours' worth of high valuable, perhaps irreplaceable data. In this case, arrange for more frequent backup that will let you hit your application-specific RPO.

Depending on application priority, individual RPOs typically range from 24 hours, to 12, to 8, to 4; down to near-zero measured in seconds. 8-hour-plus RPOs might be able to take advantage of your existing backup solution as long as it has a minimum impact on your production systems. 4-hour RPOs will need scheduled snapshot replication, and near-zero RPOs will require continuous replication. In cases where both the RPO and RTO are near-zero, combine continuous replication with failover services for near-100% application and data availability.

### How RTO and RPO Are Alike

RTO and RPO Share Several Characteristics

- Recovery time and recovery point objectives differ according to application and data priority. Even the most deep-pocket corporation cannot afford to deliver near-zero RTO or RPO for all applications, nor should they.
- The only way to assure 100% uptime (RTO) and no lost data (RPO) is by investing in failover virtual environments with continuous data replication.
- IT prioritizes applications and data to match the expense of achieving RTO and RPO. Note that priority is not only guided by revenue but also by risk. A company may use an application infrequently, but if its data is regulated then data loss may result in big fines.
- Both RTO and RPO are measured in units of time. For RTO, the metric is the amount of time that passes between application failure and full availability including data recovery. RPO is also measured in units of time. The metric is the amount of time between the loss of data and the preceding backup. For both RTO and RPO, application/data priority translates directly into shorter units of time.

### The Big Difference: Purpose

Despite their similarities, RPO and RTO serve different purposes. RTO is concerned with applications and systems. The measurement includes data recovery but primarily describes time limitations on application downtime.

RPO is concerned with the amount of data that is lost following a failure event. An annoyed user is one thing. But losing hundreds of thousands of dollars in customer transactions is far more than mere annoyance. It is catastrophic.

- https://www.enterprisestorageforum.com/storage-management/rpo-and-rto-understanding-the-differences.html
