# Consensus Service

The Consensus Service provides the ability for Hedera Hashgraph to provide aBFT consensus as to the order and validity of messages submitted to a topic, as well as a consensus timestamp for those messages.   
  
Automatic renewal can be configured via an autoRenewAccount \(not currently implemented\).   
Any time an autoRenewAccount is added to a topic, that createTopic/updateTopic transaction must be signed by the autoRenewAccount.   
  
The autoRenewPeriod on an account must currently be set a value in createTopic between MIN\_AUTORENEW\_PERIOD \(6999999seconds\) and MAX\_AUTORENEW\_PERIOD \(8000001 seconds\). During creation this sets the initial expirationTime of the topic \(see more below\).   
  
If no adminKey is on a topic, there may not be an autoRenewAccount on the topic, deleteTopic is not allowed, and the only change allowed via an updateTopic is to extend the expirationTime.   
  
If an adminKey is on a topic, every updateTopic and deleteTopic transaction must be signed by the adminKey, except for updateTopics which only extend the topic's expirationTime \(no adminKey authorization required\).   
  
If an updateTopic modifies the adminKey of a topic, the transaction signatures on the updateTopic must fulfill both the pre-update and post-update adminKey signature requirements.   
  
Mirrornet ConsensusService may be used to subscribe to changes on the topic, including changes to the topic definition and the consensus ordering and timestamp of submitted messages.   
  
Until autoRenew functionality is supported by HAPI, the topic will not expire, the autoRenewAccount will not be charged, and the topic will not automatically be deleted.

  
Once autoRenew functionality is supported by HAPI:

1. Once the expirationTime is encountered, if an autoRenewAccount is configured on the topic, the account will be charged automatically at the expirationTime, to extend the expirationTime of the topic up to the topic's autoRenewPeriod \(or as much extension as the account's balance will supply\).
2. If the topic expires and is not automatically renewed, the topic will enter the EXPIRED state. All transactions on the topic will fail with TOPIC\_EXPIRED, except an updateTopic\(\) call that modifies only the expirationTime. getTopicInfo\(\) will succeed. This state will be available for a AUTORENEW\_GRACE\_PERIOD grace period \(7 days\).
3. After the grace period, if the topic's expirationTime is not extended, the topic will be automatically deleted and no transactions or queries on the topic will succeed after that point.

## ConsensusService

| RPC | Request | Response | Comments |
| :--- | :--- | :--- | :--- |
| `createTopic` | Transaction | TransactionResponse | Create a topic to be used for consensus. If an autoRenewAccount is specified, that account must also sign this transaction. If an adminKey is specified, the adminKey must sign the transaction. On success, the resulting TransactionReceipt contains the newly created TopicId. Request is [ConsensusCreateTopicTransactionBody](consensuscreatetopic.md#consensuscreatetopictransactionbody) |
| `updateTopic` | Transaction | TransactionResponse | Update a topic. If there is no adminKey, the only authorized update \(available to anyone\) is to extend the expirationTime. Otherwise transaction must be signed by the adminKey. If an adminKey is updated, the transaction must be signed by the pre-update adminKey and post-update adminKey. If a new autoRenewAccount is specified \(not just being removed\), that account must also sign the transaction. Request is [ConsensusUpdateTopicTransactionBody](consensusupdatetopic.md#consensusupdatetopictransactionbody) |
| `deleteTopic` | Transaction | TransactionResponse | Delete a topic. No more transactions or queries on the topic \(via HAPI\) will succeed. If an adminKey is set, this transaction must be signed by that key. If there is no adminKey, this transaction will fail UNAUTHORIZED. Request is [ConsensusDeleteTopicTransactionBody](consensusdeletetopic.md) |
| `getTopicInfo` | Query | Response | Retrieve the latest state of a topic. This method is unrestricted and allowed on any topic by any payer account. Deleted accounts will not be returned. Request is [ConsensusGetTopicInfoQuery](consensusgettopicinfo.md#consensusgettopicinfoquery) Response is [ConsensusGetTopicInfoResponse](consensusgettopicinfo.md#consensusgettopicinforesponse) |
| `submitMessage` | Transaction | TransactionResponse | Submit a message for consensus. Valid and authorized messages on valid topics will be ordered by the consensus service, gossipped to the mirror net, and published \(in order\) to all subscribers \(from the mirror net\) on this topic. The submitKey \(if any\) must sign this transaction. On success, the resulting TransactionReceipt contains the topic's updated topicSequenceNumber and topicRunningHash. Request is [ConsensusSubmitMessageTransactionBody](consensussubmitmessage.md#consensussubmitmessagetransactionbody) |

