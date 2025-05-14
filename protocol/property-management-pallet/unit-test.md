# Unit Test

use crate::{mock::\*, Error};\
use frame\_support::traits::Currency;\
use frame\_support::BoundedVec;\
use frame\_support::{assert\_noop, assert\_ok};

use crate::{PropertyReserve, LettingStorage, LettingInfo, LettingAgentLocations, InvestorFunds};

use sp\_runtime::TokenError;

use pallet\_nft\_marketplace::types::{LegalProperty, PaymentAssets};

macro\_rules! bvec {\
($( $x:tt )_) => {_\
_vec!\[$( $x )_].try\_into().unwrap()\
}\
}

\#\[test]\
fn add\_letting\_agent\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_eq!(LettingInfo::::get::(\[0; 32].into()).is\_some(), true);\
let location: BoundedVec\<u8, Postcode> = bvec!\[10, 10];\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into()).unwrap().locations\[0],\
location\
);\
});\
}

\#\[test]\
fn add\_letting\_agent\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_noop!(\
PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
),\
Error::::RegionUnknown\
);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_noop!(\
PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
),\
Error::::LocationUnknown\
);\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_eq!(LettingInfo::::get::(\[0; 32].into()).is\_some(), true);\
assert\_noop!(\
PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
),\
Error::::LettingAgentExists\
);\
});\
}

\#\[test]\
fn let\_letting\_agent\_deposit() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.contains(&\[0; 32].into()),\
false\
);\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into()).unwrap().deposited,\
false\
);\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.contains(&\[0; 32].into()),\
true\
);\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into()).unwrap().deposited,\
true\
);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
});\
}

\#\[test]\
fn let\_letting\_agent\_deposit\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_noop!(\
PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\[0; 32].into())),\
Error::::AlreadyDeposited\
);\
assert\_noop!(\
PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\[1; 32].into())),\
Error::::NoPermission\
);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
for x in 1..100 {\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[x; 32].into(),\
));\
Balances::make\_free\_balance\_be(&\[x; 32].into(), 200);\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[x; 32].into()\
)));\
}\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[100; 32].into(),\
));\
Balances::make\_free\_balance\_be(&\[100; 32].into(), 200);\
assert\_noop!(\
PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\[100; 32].into())),\
Error::::TooManyLettingAgents\
);\
});\
}

\#\[test]\
fn let\_letting\_agent\_deposit\_not\_enough\_funds() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[5; 32].into(),\
));\
assert\_noop!(\
PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\[5; 32].into())),\
TokenError::FundsUnavailable\
);\
});\
}

\#\[test]\
fn add\_letting\_agent\_to\_location\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[9, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[9, 10],\
\[0; 32].into(),\
));\
assert\_eq!(LettingInfo::::get::(\[0; 32].into()).is\_some(), true);\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent\_to\_location(\
RuntimeOrigin::root(),\
bvec!\[10, 10],\
\[0; 32].into()\
));\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[9, 10]\
)\
.contains(&\[0; 32].into()),\
true\
);\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.contains(&\[0; 32].into()),\
true\
);\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into())\
.unwrap()\
.locations\
.len(),\
2\
);\
});\
}

\#\[test]\
fn add\_letting\_agent\_to\_location\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_noop!(\
PropertyManagement::add\_letting\_agent\_to\_location(\
RuntimeOrigin::root(),\
bvec!\[10, 10],\
\[0; 32].into()\
),\
Error::::NoLettingAgentFound\
);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[9, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_eq!(LettingInfo::::get::(\[0; 32].into()).is\_some(), true);\
assert\_noop!(\
PropertyManagement::add\_letting\_agent\_to\_location(\
RuntimeOrigin::root(),\
bvec!\[10, 10],\
\[0; 32].into()\
),\
Error::::NotDeposited\
);\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_noop!(\
PropertyManagement::add\_letting\_agent\_to\_location(\
RuntimeOrigin::root(),\
bvec!\[5, 10],\
\[0; 32].into()\
),\
Error::::LocationUnknown\
);\
assert\_noop!(\
PropertyManagement::add\_letting\_agent\_to\_location(\
RuntimeOrigin::root(),\
bvec!\[10, 10],\
\[0; 32].into()\
),\
Error::::LettingAgentInLocation\
);\
});\
}

\#\[test]\
fn set\_letting\_agent\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
1\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 1, 100, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
1\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 2, 100, PaymentAssets::USDT));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[2; 32].into(),\
));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[3; 32].into(),\
));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[4; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[2; 32].into()\
)));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[3; 32].into()\
)));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[4; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[2; 32].into()), 0));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[3; 32].into()), 1));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[4; 32].into()), 2));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
1\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 3, 100, PaymentAssets::USDT));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[2; 32].into()), 3));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[2; 32].into());\
assert\_eq!(LettingStorage::::get(1).unwrap(), \[3; 32].into());\
assert\_eq!(LettingStorage::::get(2).unwrap(), \[4; 32].into());\
assert\_eq!(LettingStorage::::get(3).unwrap(), \[2; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
3\
);\
assert\_eq!(\
LettingInfo::::get::(\[2; 32].into())\
.unwrap()\
.assigned\_properties\
.len(),\
2\
);\
assert\_eq!(\
LettingInfo::::get::(\[3; 32].into())\
.unwrap()\
.assigned\_properties\
.len(),\
1\
);\
assert\_eq!(\
LettingInfo::::get::(\[4; 32].into())\
.unwrap()\
.assigned\_properties\
.len(),\
1\
);\
});\
}

\#\[test]\
fn set\_letting\_agent\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
assert\_noop!(\
PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0),\
Error::::NoObjectFound\
);\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
100,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_noop!(\
PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0),\
Error::::LettingAgentAlreadySet\
);\
for x in 1..100 {\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
1\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[(x + 1); 32].into()));\
Balances::make\_free\_balance\_be(&\[x + 1; 32].into(), 100\_000);\
assert\_ok!(ForeignAssets::mint(\
RuntimeOrigin::signed(\[0; 32].into()),\
1984.into(),\
sp\_runtime::MultiAddress::Id(\[(x + 1); 32].into()),\
1\_000\_000,\
));\
assert\_ok!(NftMarketplace::buy\_token(\
RuntimeOrigin::signed(\[(x + 1); 32].into()),\
(x as u32).into(),\
100,\
PaymentAssets::USDT\
));\
assert\_ok!(PropertyManagement::set\_letting\_agent(\
RuntimeOrigin::signed(\[0; 32].into()),\
x.into()\
));\
}\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 100, 100, PaymentAssets::USDT));\
assert\_noop!(\
PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 100),\
Error::::TooManyAssignedProperties\
);\
});\
}

\#\[test]\
fn set\_letting\_agent\_no\_letting\_agent() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_noop!(\
PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0),\
Error::::AgentNotFound\
);\
});\
}

\#\[test]\
fn distribute\_income\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
9\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 50, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[4; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[4; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[4; 32].into()), 0));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3200,\
PaymentAssets::USDT,\
));\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 40);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[2; 32].into(), 0, PaymentAssets::USDT)), 60);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[3; 32].into(), 0, PaymentAssets::USDT)), 100);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 1800);\
});\
}

\#\[test]\
fn distribute\_income\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
assert\_noop!(\
PropertyManagement::distribute\_income(RuntimeOrigin::signed(\[5; 32].into()), 0, 200, PaymentAssets::USDT),\
Error::::NoLettingAgentFound\
);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[4; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[4; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[4; 32].into()), 0));\
assert\_noop!(\
PropertyManagement::distribute\_income(RuntimeOrigin::signed(\[5; 32].into()), 0, 200, PaymentAssets::USDT),\
Error::::NoPermission\
);\
assert\_noop!(\
PropertyManagement::distribute\_income(RuntimeOrigin::signed(\[4; 32].into()), 0, 20000, PaymentAssets::USDT),\
Error::::NotEnoughFunds\
);\
});\
}

\#\[test]\
fn withdraw\_funds\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
9\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[4; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[4; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[4; 32].into()), 0));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
2200,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDC,\
));\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDC)), 200);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2800);\
assert\_eq!(Balances::free\_balance(&(\[4; 32].into())), 4900);\
assert\_eq!(Balances::free\_balance(\&PropertyManagement::property\_account\_id(0)), 5085);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyManagement::property\_account\_id(0)), 2200);\
assert\_eq!(ForeignAssets::balance(1337, \&PropertyManagement::property\_account\_id(0)), 1000);\
assert\_ok!(PropertyManagement::withdraw\_funds(RuntimeOrigin::signed(\[1; 32].into()), 0, PaymentAssets::USDC));\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyManagement::property\_account\_id(0)), 2200);\
assert\_eq!(ForeignAssets::balance(1337, \&PropertyManagement::property\_account\_id(0)), 800);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 564\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[1; 32].into()), 200);\
});\
}

\#\[test]\
fn withdraw\_funds\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
900,\
1000,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 1000, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_eq!(LocalAssets::total\_supply(0), 1000);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[4; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[4; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[4; 32].into()), 0));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3200,\
PaymentAssets::USDT,\
));\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 200);\
assert\_noop!(\
PropertyManagement::withdraw\_funds(RuntimeOrigin::signed(\[2; 32].into()), 0, PaymentAssets::USDT),\
Error::::UserHasNoFundsStored\
);\
});\
}
