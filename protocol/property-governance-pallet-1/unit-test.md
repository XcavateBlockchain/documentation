# Unit Test

use crate::{mock::\*, Error, Event};\
use frame\_support::{\
assert\_noop, assert\_ok,\
traits::{OnFinalize, OnInitialize},\
BoundedVec, sp\_runtime::Percent\
};

use crate::{Proposals, Challenges, ChallengeRoundsExpiring, OngoingChallengeVotes, OngoingVotes};

use pallet\_property\_management::{\
PropertyReserve, LettingStorage, PropertyDebts, InvestorFunds,\
LettingAgentLocations, LettingInfo\
};

use pallet\_nft\_marketplace::types::{LegalProperty, PaymentAssets};

macro\_rules! bvec {\
($( $x:tt )_) => {_\
_vec!\[$( $x )_].try\_into().unwrap()\
}\
}

fn run\_to\_block(n: u64) {\
while System::block\_number() < n {\
if System::block\_number() > 0 {\
PropertyGovernance::on\_finalize(System::block\_number());\
System::on\_finalize(System::block\_number());\
}\
System::reset\_events();\
System::set\_block\_number(System::block\_number() + 1);\
System::on\_initialize(System::block\_number());\
PropertyGovernance::on\_initialize(System::block\_number());\
}\
}

\#\[test]\
fn propose\_works() {\
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
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[2; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[2; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[2; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[2; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(1).unwrap().asset\_id, 0);\
assert\_eq!(OngoingVotes::::get(1).is\_some(), true);\
});\
}

\#\[test]\
fn proposal\_with\_low\_amount\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
500,\
bvec!\[10, 10]\
));\
assert\_eq!(Balances::free\_balance(&(\[4; 32].into())), 4900);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4500);\
assert\_eq!(OngoingVotes::::get(1).is\_some(), false);\
});\
}

\#\[test]\
fn propose\_fails() {\
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
PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
),\
Error::::NoLettingAgentFound\
);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_noop!(\
PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn challenge\_against\_letting\_agent\_works() {\
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
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).is\_some(), true);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::First);\
});\
}

\#\[test]\
fn challenge\_against\_letting\_agent\_fails() {\
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
10\_000,\
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
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
), Error::::NoLettingAgentFound);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[2; 32].into()),\
0\
), Error::::NoPermission);\
assert\_eq!(Challenges::::get(1).is\_some(), false);\
});\
}

\#\[test]\
fn vote\_on\_proposal\_works() {\
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
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 10, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 40, PaymentAssets::USDT));\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(OngoingVotes::::get(1).unwrap().yes\_voting\_power, 10);\
assert\_eq!(OngoingVotes::::get(1).unwrap().no\_voting\_power, 90);\
});\
}

\#\[test]\
fn proposal\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 19\_999\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1\_000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(OngoingVotes::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn proposal\_pass\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
10000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_eq!(PropertyReserve::::get(0).usdt, 1000);\
assert\_eq!(PropertyReserve::::get(0).usdc, 0);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ProposalExecuted{ asset\_id: 0, amount: 10000}.into());\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(PropertyDebts::::get(0), 9\_000);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
PaymentAssets::USDT,\
));\
assert\_eq!(PropertyDebts::::get(0), 6000);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
});\
}

\#\[test]\
fn proposal\_not\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
System::assert\_last\_event(Event::ProposalRejected{ proposal\_id: 1}.into());\
});\
}

\#\[test]\
fn proposal\_not\_pass\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 40, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
10000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(Proposals::::get(1).unwrap().amount, 10000);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ProposalThresHoldNotReached{ proposal\_id: 1, required\_threshold: Percent::from\_percent(67)}.into());\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
});\
}

\#\[test]\
fn vote\_on\_proposal\_fails() {\
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
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_noop!(\
PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NotOngoing\
);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_noop!(\
PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn vote\_on\_challenge\_works() {\
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
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 10, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 40, PaymentAssets::USDT));\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[3; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(OngoingChallengeVotes::::get(1, crate::ChallengeState::First).unwrap().yes\_voting\_power, 60);\
assert\_eq!(OngoingChallengeVotes::::get(1, crate::ChallengeState::First).unwrap().no\_voting\_power, 40);\
});\
}

\#\[test]\
fn challenge\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 70, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Fourth);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into())\
.unwrap()\
.locations\
.len(),\
1\
);\
run\_to\_block(121);\
assert\_eq!(LettingStorage::::get(0).is\_none(), true);\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into())\
.unwrap()\
.locations\
.len(),\
1\
);\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[1; 32].into());\
});\
}

\#\[test]\
fn challenge\_does\_not\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
4\_000,\
250,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 75, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 175, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
System::assert\_last\_event(Event::ChallengeThresHoldNotReached{ challenge\_id: 1, required\_threshold: Percent::from\_percent(51), challenge\_state: crate::ChallengeState::Third}.into());\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn challenge\_pass\_only\_one\_agent() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[9, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[9, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 70, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
1\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Fourth);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
run\_to\_block(121);\
assert\_eq!(LettingStorage::::get(0).is\_none(), true);\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
1\
);\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn challenge\_not\_pass() {\
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
10\_000,\
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
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
), Error::::NoLettingAgentFound);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(Challenges::::get(1).is\_some(), true);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ChallengeRejected{ challenge\_id: 1, challenge\_state: crate::ChallengeState::First}.into());\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn vote\_on\_challenge\_fails() {\
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
10\_000,\
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
assert\_noop!(\
PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NotOngoing\
);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_noop!(\
PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn different\_proposals() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
5\_000,\
200,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 80, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(2).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
2,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
2,\
crate::Vote::Yes\
));\
run\_to\_block(61);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(3).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
3,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
3,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
3,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1700,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
300,\
PaymentAssets::USDC,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1500,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(4).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
4,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
4,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
4,\
crate::Vote::No\
));\
run\_to\_block(121);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4800);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 200);\
assert\_eq!(ForeignAssets::balance(1337, &\[4; 32].into()), 4700);\
assert\_eq!(ForeignAssets::balance(1337, \&PropertyGovernance::property\_account\_id(0)), 300);\
assert\_eq!(PropertyReserve::::get(0).total, 500);\
});\
}use crate::{mock::\*, Error, Event};\
use frame\_support::{\
assert\_noop, assert\_ok,\
traits::{OnFinalize, OnInitialize},\
BoundedVec, sp\_runtime::Percent\
};

use crate::{Proposals, Challenges, ChallengeRoundsExpiring, OngoingChallengeVotes, OngoingVotes};

use pallet\_property\_management::{\
PropertyReserve, LettingStorage, PropertyDebts, InvestorFunds,\
LettingAgentLocations, LettingInfo\
};

use pallet\_nft\_marketplace::types::{LegalProperty, PaymentAssets};

macro\_rules! bvec {\
($( $x:tt )_) => {_\
_vec!\[$( $x )_].try\_into().unwrap()\
}\
}

fn run\_to\_block(n: u64) {\
while System::block\_number() < n {\
if System::block\_number() > 0 {\
PropertyGovernance::on\_finalize(System::block\_number());\
System::on\_finalize(System::block\_number());\
}\
System::reset\_events();\
System::set\_block\_number(System::block\_number() + 1);\
System::on\_initialize(System::block\_number());\
PropertyGovernance::on\_initialize(System::block\_number());\
}\
}

\#\[test]\
fn propose\_works() {\
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
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[2; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[2; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[2; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[2; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(1).unwrap().asset\_id, 0);\
assert\_eq!(OngoingVotes::::get(1).is\_some(), true);\
});\
}

\#\[test]\
fn proposal\_with\_low\_amount\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
500,\
bvec!\[10, 10]\
));\
assert\_eq!(Balances::free\_balance(&(\[4; 32].into())), 4900);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4500);\
assert\_eq!(OngoingVotes::::get(1).is\_some(), false);\
});\
}

\#\[test]\
fn propose\_fails() {\
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
PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
),\
Error::::NoLettingAgentFound\
);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_noop!(\
PropertyGovernance::propose(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn challenge\_against\_letting\_agent\_works() {\
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
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).is\_some(), true);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::First);\
});\
}

\#\[test]\
fn challenge\_against\_letting\_agent\_fails() {\
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
10\_000,\
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
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
), Error::::NoLettingAgentFound);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[2; 32].into()),\
0\
), Error::::NoPermission);\
assert\_eq!(Challenges::::get(1).is\_some(), false);\
});\
}

\#\[test]\
fn vote\_on\_proposal\_works() {\
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
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 10, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 40, PaymentAssets::USDT));\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(OngoingVotes::::get(1).unwrap().yes\_voting\_power, 10);\
assert\_eq!(OngoingVotes::::get(1).unwrap().no\_voting\_power, 90);\
});\
}

\#\[test]\
fn proposal\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 19\_999\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1\_000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(OngoingVotes::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn proposal\_pass\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
10000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_eq!(PropertyReserve::::get(0).usdt, 1000);\
assert\_eq!(PropertyReserve::::get(0).usdc, 0);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ProposalExecuted{ asset\_id: 0, amount: 10000}.into());\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(PropertyDebts::::get(0), 9\_000);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
PaymentAssets::USDT,\
));\
assert\_eq!(PropertyDebts::::get(0), 6000);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_eq!(InvestorFunds::::get::<(AccountId, u32, PaymentAssets)>((\[1; 32].into(), 0, PaymentAssets::USDT)), 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
});\
}

\#\[test]\
fn proposal\_not\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(Balances::free\_balance(&(\[0; 32].into())), 19\_999\_900);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
System::assert\_last\_event(Event::ProposalRejected{ proposal\_id: 1}.into());\
});\
}

\#\[test]\
fn proposal\_not\_pass\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 40, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
10000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(Proposals::::get(1).unwrap().amount, 10000);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ProposalThresHoldNotReached{ proposal\_id: 1, required\_threshold: Percent::from\_percent(67)}.into());\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 1000);\
assert\_eq!(PropertyReserve::::get(0).total, 1000);\
});\
}

\#\[test]\
fn vote\_on\_proposal\_fails() {\
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
10\_000,\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_noop!(\
PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NotOngoing\
);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_noop!(\
PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn vote\_on\_challenge\_works() {\
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
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 20, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 10, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 40, PaymentAssets::USDT));\
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
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[3; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(OngoingChallengeVotes::::get(1, crate::ChallengeState::First).unwrap().yes\_voting\_power, 60);\
assert\_eq!(OngoingChallengeVotes::::get(1, crate::ChallengeState::First).unwrap().no\_voting\_power, 40);\
});\
}

\#\[test]\
fn challenge\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 70, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Fourth);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into())\
.unwrap()\
.locations\
.len(),\
1\
);\
run\_to\_block(121);\
assert\_eq!(LettingStorage::::get(0).is\_none(), true);\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_eq!(\
LettingInfo::::get::(\[0; 32].into())\
.unwrap()\
.locations\
.len(),\
1\
);\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[1; 32].into());\
});\
}

\#\[test]\
fn challenge\_does\_not\_pass() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
4\_000,\
250,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 75, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 175, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
2\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
System::assert\_last\_event(Event::ChallengeThresHoldNotReached{ challenge\_id: 1, required\_threshold: Percent::from\_percent(51), challenge\_state: crate::ChallengeState::Third}.into());\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn challenge\_pass\_only\_one\_agent() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[9, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[9, 10],\
\[1; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[1; 32].into()\
)));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 70, PaymentAssets::USDT));\
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
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
1\
);\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_eq!(Challenges::::get(1).unwrap().asset\_id, 0);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(ChallengeRoundsExpiring::::get(31).len(), 1);\
run\_to\_block(31);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Second);\
run\_to\_block(61);\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Third);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(Challenges::::get(1).unwrap().state, crate::ChallengeState::Fourth);\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(LettingStorage::::get(0).unwrap(), \[0; 32].into());\
run\_to\_block(121);\
assert\_eq!(LettingStorage::::get(0).is\_none(), true);\
assert\_eq!(\
LettingAgentLocations::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
)\
.len(),\
1\
);\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn challenge\_not\_pass() {\
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
10\_000,\
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
assert\_noop!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
), Error::::NoLettingAgentFound);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::No\
));\
assert\_eq!(Challenges::::get(1).is\_some(), true);\
run\_to\_block(31);\
System::assert\_last\_event(Event::ChallengeRejected{ challenge\_id: 1, challenge\_state: crate::ChallengeState::First}.into());\
assert\_eq!(Challenges::::get(1).is\_none(), true);\
});\
}

\#\[test]\
fn vote\_on\_challenge\_fails() {\
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
10\_000,\
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
assert\_noop!(\
PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NotOngoing\
);\
assert\_ok!(PropertyManagement::add\_letting\_agent(\
RuntimeOrigin::root(),\
0,\
bvec!\[10, 10],\
\[0; 32].into(),\
));\
assert\_ok!(PropertyManagement::letting\_agent\_deposit(RuntimeOrigin::signed(\
\[0; 32].into()\
)));\
assert\_ok!(PropertyManagement::set\_letting\_agent(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_ok!(PropertyGovernance::challenge\_against\_letting\_agent(\
RuntimeOrigin::signed(\[1; 32].into()),\
0\
));\
assert\_ok!(PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_noop!(\
PropertyGovernance::vote\_on\_letting\_agent\_challenge(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
crate::Vote::Yes\
),\
Error::::NoPermission\
);\
});\
}

\#\[test]\
fn different\_proposals() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[6; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
5\_000,\
200,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 60, PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 80, PaymentAssets::USDT));\
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
assert\_eq!(LettingStorage::::get(0).unwrap(), \[4; 32].into());\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1000,\
bvec!\[10, 10]\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
crate::Vote::Yes\
));\
assert\_eq!(Proposals::::get(1).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
run\_to\_block(31);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_eq!(Proposals::::get(1).is\_none(), true);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(2).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
2,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
2,\
crate::Vote::Yes\
));\
run\_to\_block(61);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 2000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 3000);\
assert\_eq!(PropertyReserve::::get(0).total, 3000);\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
3000,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(3).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
3,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
3,\
crate::Vote::No\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
3,\
crate::Vote::Yes\
));\
run\_to\_block(91);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 5000);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 0);\
assert\_eq!(PropertyReserve::::get(0).total, 0);\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1700,\
PaymentAssets::USDT,\
));\
assert\_ok!(PropertyManagement::distribute\_income(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
300,\
PaymentAssets::USDC,\
));\
assert\_ok!(PropertyGovernance::propose(\
RuntimeOrigin::signed(\[4; 32].into()),\
0,\
1500,\
bvec!\[10, 10]\
));\
assert\_eq!(Proposals::::get(4).is\_some(), true);\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[1; 32].into()),\
4,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[2; 32].into()),\
4,\
crate::Vote::Yes\
));\
assert\_ok!(PropertyGovernance::vote\_on\_proposal(\
RuntimeOrigin::signed(\[3; 32].into()),\
4,\
crate::Vote::No\
));\
run\_to\_block(121);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 4800);\
assert\_eq!(ForeignAssets::balance(1984, \&PropertyGovernance::property\_account\_id(0)), 200);\
assert\_eq!(ForeignAssets::balance(1337, &\[4; 32].into()), 4700);\
assert\_eq!(ForeignAssets::balance(1337, \&PropertyGovernance::property\_account\_id(0)), 300);\
assert\_eq!(PropertyReserve::::get(0).total, 500);\
});\
}
