# Unit Test

use crate::{mock::\*, \*, Error};\
use frame\_support::BoundedVec;\
use frame\_support::{assert\_noop, assert\_ok, traits::{OnFinalize, OnInitialize, fungible::InspectHold\}};\
use crate::{RegionCollections, LocationRegistration, ListedToken, NextNftId,\
OngoingObjectListing, NextAssetId, RegisteredNftDetails, TokenOwner, TokenBuyer,\
TokenListings, OngoingOffers, PropertyOwnerToken, PropertyOwner, PropertyLawyer,\
RealEstateLawyer, RefundToken};\
use pallet\_assets::FrozenBalance;\
use sp\_runtime::TokenError;

macro\_rules! bvec {\
($( $x:tt )_) => {_\
_vec!\[$( $x )_].try\_into().unwrap()\
}\
}

fn run\_to\_block(n: u64) {\
while System::block\_number() < n {\
if System::block\_number() > 0 {\
NftMarketplace::on\_finalize(System::block\_number());\
System::on\_finalize(System::block\_number());\
}\
System::reset\_events();\
System::set\_block\_number(System::block\_number() + 1);\
System::on\_initialize(System::block\_number());\
NftMarketplace::on\_initialize(System::block\_number());\
}\
}

// create\_new\_region function\
\#\[test]\
fn create\_new\_region\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_eq!(Balances::free\_balance(&(\[8; 32].into())), 200\_000);\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::RegionDepositReserve.into(), &(\[8; 32].into())), 200\_000);\
assert\_eq!(RegionCollections::::get(0).unwrap(), 0);\
assert\_eq!(RegionCollections::::get(1).unwrap(), 1);\
})\
}

\#\[test]\
fn create\_new\_region\_does\_not\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_noop!(\
NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[7; 32].into()), 30),\
TokenError::FundsUnavailable\
);\
})\
}

// create\_new\_location function\
\#\[test]\
fn create\_new\_location\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[9, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 1, bvec!\[9, 10]));\
assert\_eq!(\
LocationRegistration::::get::\<u32, BoundedVec\<u8, Postcode>>(\
0,\
bvec!\[10, 10]\
),\
true\
);\
assert\_eq!(\
LocationRegistration::::get::\<u32, BoundedVec\<u8, Postcode>>(0, bvec!\[9, 10]),\
true\
);\
assert\_eq!(\
LocationRegistration::::get::\<u32, BoundedVec\<u8, Postcode>>(1, bvec!\[9, 10]),\
true\
);\
assert\_eq!(\
LocationRegistration::::get::\<u32, BoundedVec\<u8, Postcode>>(\
1,\
bvec!\[10, 10]\
),\
false\
);\
assert\_eq!(\
LocationRegistration::::get::\<u32, BoundedVec\<u8, Postcode>>(1, bvec!\[8, 10]),\
false\
);\
})\
}

\#\[test]\
fn create\_new\_location\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_noop!(\
NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 1, bvec!\[10, 10]),\
Error::::RegionUnknown\
);\
})\
}

// register\_lawyer function\
\#\[test]\
fn register\_lawyer\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_eq!(RealEstateLawyer::::get::(\[0; 32].into()), false);\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_eq!(RealEstateLawyer::::get::(\[0; 32].into()), true);\
})\
}

\#\[test]\
fn register\_lawyer\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_noop!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[0; 32].into()), Error::::LawyerAlreadyRegistered);\
})\
}

// list\_object function\
\#\[test]\
fn list\_object\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 100\_000);\
assert\_eq!(ListedToken::::get(0).unwrap(), 100);\
assert\_eq!(NextNftId::::get(0), 1);\
assert\_eq!(NextNftId::::get(1), 0);\
assert\_eq!(NextAssetId::::get(), 1);\
assert\_eq!(OngoingObjectListing::::get(0).is\_some(), true);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).is\_some(), true);\
assert\_eq!(Nfts::owner(0, 0).unwrap(), NftMarketplace::property\_account\_id(0));\
})\
}

\#\[test]\
fn list\_object\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_noop!(\
NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
),\
Error::::RegionUnknown\
);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_noop!(\
NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
),\
Error::::LocationUnknown\
);\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_noop!(\
NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
251,\
bvec!\[22, 22]\
),\
Error::::TooManyToken\
);\
assert\_noop!(\
NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
0,\
bvec!\[22, 22]\
),\
Error::::AmountCannotBeZero\
);\
})\
}

// buy\_token function\
\#\[test]\
fn buy\_token\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[6; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[14; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[14; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000\_000\_000\_000\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[6; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_eq!(ListedToken::::get(0).unwrap(), 70);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[6; 32].into(), 0).token\_amount, 30);\
assert\_eq!(TokenBuyer::::get(0).len(), 1);\
assert\_eq!(Balances::free\_balance(&(\[6; 32].into())), 5\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[6; 32].into()), 1\_500\_000\_000\_000\_000\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[6; 32].into()), Some(312\_000\_000\_000\_000\_000));\
})\
}

\#\[test]\
fn buy\_token\_doesnt\_work() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_noop!(\
NftMarketplace::buy\_token(RuntimeOrigin::signed(\[0; 32].into()), 1, 1, crate::PaymentAssets::USDT),\
Error::::TokenNotForSale\
);\
})\
}

\#\[test]\
fn buy\_token\_doesnt\_work\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_noop!(\
NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 101, crate::PaymentAssets::USDT),\
Error::::NotEnoughTokenAvailable\
);\
run\_to\_block(32);\
assert\_noop!(\
NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT),\
Error::::ListingExpired\
);\
})\
}

\#\[test]\
fn buy\_token\_fails\_insufficient\_balance() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[14; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[4; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[14; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000\_000\_000\_000\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_noop!(\
NftMarketplace::buy\_token(RuntimeOrigin::signed(\[4; 32].into()), 0, 30, crate::PaymentAssets::USDT),\
Error::::NotEnoughFunds\
);\
assert\_eq!(ForeignAssets::balance(1984, &\[4; 32].into()), 50);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[6; 32].into()), None);\
})\
}

\#\[test]\
fn listing\_and\_selling\_multiple\_objects() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[15; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[15; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 1, 80, crate::PaymentAssets::USDT));\
assert\_eq!(PropertyLawyer::::get(1).is\_some(), false);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 1, 20, crate::PaymentAssets::USDT));\
assert\_eq!(PropertyLawyer::::get(1).is\_some(), true);\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
1,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
1,\
true,\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 2, 10, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 2, 10, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 2, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[15; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 33, crate::PaymentAssets::USDT));\
assert\_eq!(ListedToken::::get(0).unwrap(), 67);\
assert\_eq!(ListedToken::::get(2).unwrap(), 50);\
assert\_eq!(ListedToken::::get(3).unwrap(), 100);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[2; 32].into(), 2).token\_amount, 30);\
assert\_eq!(TokenBuyer::::get(2).len(), 2);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 1).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(1).len(), 0);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(1, \[1; 32].into()), 100);\
});\
}

// lawyer\_claim\_property function\
\#\[test]\
fn claim\_property\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
})\
}

\#\[test]\
fn claim\_property\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 99, crate::PaymentAssets::USDT));\
assert\_noop!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
), Error::::InvalidIndex);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 1, crate::PaymentAssets::USDT));\
assert\_noop!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[9; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
), Error::::NoPermission);\
assert\_noop!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
11\_000,\
), Error::::CostsTooHigh);\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_noop!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
), Error::::LawyerJobTaken);\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, None);\
})\
}

// remove\_from\_case function\
\#\[test]\
fn remove\_from\_case\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[12; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::remove\_from\_case(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, None);\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[12; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[12; 32].into()));\
})\
}

\#\[test]\
fn remove\_from\_case\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_noop!(NftMarketplace::remove\_from\_case(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
), Error::::NoPermission);\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_noop!(NftMarketplace::remove\_from\_case(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
), Error::::InvalidIndex);\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_noop!(NftMarketplace::remove\_from\_case(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
), Error::::AlreadyConfirmed);\
})\
}

// lawyer\_confirm\_documents function\
\#\[test]\
fn distributes\_nfts\_and\_funds() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 100\_000);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 40, crate::PaymentAssets::USDC));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Approved);\
assert\_eq!(LocalAssets::balance(0, \&NftMarketplace::property\_account\_id(0)), 0);\
assert\_eq!(OngoingObjectListing::::get(0).unwrap().asset\_id, 0);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 100);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 0);\
assert\_eq!(PropertyLawyer::::get(1).is\_some(), false);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_594\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[0; 32].into()), 20\_396\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 4\_000);\
assert\_eq!(ForeignAssets::balance(1337, \&NftMarketplace::treasury\_account\_id()), 8\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 876\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[10; 32].into()), 22\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[11; 32].into()), 4\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[1; 32].into()), 1\_084\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[10; 32].into()), 12\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[11; 32].into()), 0);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_eq!(ListedToken::::get(0), None);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(0).len(), 0);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 100);\
})\
}

\#\[test]\
fn distributes\_nfts\_and\_funds\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
//assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 00, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDC));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Approved);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_eq!(PropertyLawyer::::get(1).is\_some(), false);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[0; 32].into()), 20\_990\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 0);\
assert\_eq!(ForeignAssets::balance(1337, \&NftMarketplace::treasury\_account\_id()), 12000);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[10; 32].into()), 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[11; 32].into()), 0);\
assert\_eq!(ForeignAssets::balance(1337, &\[1; 32].into()), 460\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[10; 32].into()), 34\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[11; 32].into()), 4\_000);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_eq!(ListedToken::::get(0), None);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(0).len(), 0);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 100);\
})\
}

\#\[test]\
fn reject\_contract\_and\_refund() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 60, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 40, crate::PaymentAssets::USDC));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
false,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Rejected);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), false);

```
	assert_eq!(ForeignAssetsFreezer::frozen_balance(1984, &[1; 32].into()), None);
	assert_eq!(ForeignAssetsFreezer::frozen_balance(1337, &[1; 32].into()), None);
	assert_eq!(ForeignAssets::balance(1984, &[1; 32].into()), 876_000);
	assert_eq!(ForeignAssets::balance(1337, &[1; 32].into()), 1_084_000);
	assert_eq!(
		TokenOwner::<Test>::get::<AccountId, u32>([1; 32].into(), 0)
			.paid_funds
			.get(&crate::PaymentAssets::USDT)
			.unwrap(),
		&600000_u128
	);
	assert_eq!(
		TokenOwner::<Test>::get::<AccountId, u32>([1; 32].into(), 0)
			.paid_tax
			.get(&crate::PaymentAssets::USDT)
			.unwrap(),
		&18000_u128
	); 
	assert_ok!(NftMarketplace::lawyer_confirm_documents(
		RuntimeOrigin::signed([11; 32].into()),
		0,
		false,
	));
	assert_eq!(RefundToken::<Test>::get(0).unwrap().refund_amount, 100);
	assert_eq!(LocalAssets::balance(0, &[1; 32].into()), 100);
	assert_eq!(TokenOwner::<Test>::get::<AccountId, u32>([1; 32].into(), 0).token_amount, 100);
	assert_eq!(Balances::balance_on_hold(&HoldReason::ListingDepositReserve.into(), &([0; 32].into())), 100_000);
	assert_ok!(NftMarketplace::withdraw_funds(RuntimeOrigin::signed([1; 32].into()), 0));
	assert_eq!(Balances::balance_on_hold(&HoldReason::ListingDepositReserve.into(), &([0; 32].into())), 0);
	assert_eq!(RefundToken::<Test>::get(0).is_none(), true);
	assert_eq!(PropertyLawyer::<Test>::get(1).is_some(), false);
	assert_eq!(ForeignAssets::balance(1984, &[0; 32].into()), 20_000_000);
	assert_eq!(ForeignAssets::balance(1984, &NftMarketplace::treasury_account_id()), 2000);
	assert_eq!(ForeignAssets::balance(1337, &NftMarketplace::treasury_account_id()), 4000); 
	assert_eq!(ForeignAssets::balance(1984, &NftMarketplace::property_account_id(0)), 0);
	assert_eq!(ForeignAssets::balance(1984, &[1; 32].into()), 1_494_000);
	assert_eq!(ForeignAssets::balance(1337, &[1; 32].into()), 1_496_000);
	assert_eq!(ForeignAssets::balance(1984, &[11; 32].into()), 4000);
	assert_eq!(ForeignAssets::balance(1337, &[11; 32].into()), 0);
	assert_eq!(RegisteredNftDetails::<Test>::get(0, 0).is_none(), true);
	assert_eq!(ListedToken::<Test>::get(0), None);
	assert_eq!(TokenOwner::<Test>::get::<AccountId, u32>([1; 32].into(), 0).token_amount, 0);
	assert_eq!(TokenBuyer::<Test>::get(0).len(), 0);
	assert_eq!(pallet_nfts::Item::<Test>::get(0, 0).is_none(), true);
	assert_eq!(ForeignAssets::balance(1984, &NftMarketplace::property_account_id(0)), 0);
	assert_eq!(Balances::free_balance(&(NftMarketplace::property_account_id(0))), 0);
	assert_eq!(Balances::balance(&(NftMarketplace::property_account_id(0))), 0);
})
```

}

\#\[test]\
fn reject\_contract\_and\_refund\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[7; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[7; 32].into()), 0, 40, crate::PaymentAssets::USDC));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
false,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Rejected);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), false);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_188\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[2; 32].into()), 838\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[7; 32].into()), 84\_000);\
assert\_eq!(ForeignAssets::balance(1337, &\[1; 32].into()), 1\_500\_000);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
false,\
));\
assert\_eq!(RefundToken::::get(0).unwrap().refund\_amount, 100);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 30);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 30);\
assert\_ok!(NftMarketplace::withdraw\_funds(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(RefundToken::::get(0).unwrap().refund\_amount, 70);\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_497\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[11; 32].into()), 0);\
assert\_eq!(ForeignAssets::balance(1337, &\[11; 32].into()), 0);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(0).len(), 3);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), false);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::property\_account\_id(0)), 315000);\
assert\_ok!(NftMarketplace::withdraw\_funds(RuntimeOrigin::signed(\[2; 32].into()), 0));\
assert\_ok!(NftMarketplace::withdraw\_funds(RuntimeOrigin::signed(\[7; 32].into()), 0));\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), true);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::property\_account\_id(0)), 0);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 2000);\
assert\_eq!(ForeignAssets::balance(1337, \&NftMarketplace::treasury\_account\_id()), 4000);\
assert\_eq!(ForeignAssets::balance(1984, &\[11; 32].into()), 4000);\
assert\_eq!(RefundToken::::get(0).is\_none(), true);\
assert\_eq!(TokenBuyer::::get(0).len(), 0);\
})\
}

\#\[test]\
fn second\_attempt\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Approved);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), false);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
false,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().second\_attempt, true);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
false,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_status, crate::DocumentStatus::Rejected);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), false);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
true,\
));\
assert\_ok!(NftMarketplace::withdraw\_funds(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(ForeignAssets::balance(1984, &\[0; 32].into()), 20\_000\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 6000);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_490\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[11; 32].into()), 4\_000);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).is\_none(), true);\
assert\_eq!(ListedToken::::get(0), None);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(0).len(), 0);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), true);\
})\
}

\#\[test]\
fn lawyer\_confirm\_documents\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[10; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[11; 32].into()));\
assert\_ok!(NftMarketplace::register\_lawyer(RuntimeOrigin::root(), \[12; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().real\_estate\_developer\_lawyer, Some(\[10; 32].into()));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_eq!(PropertyLawyer::::get(0).unwrap().spv\_lawyer, Some(\[11; 32].into()));\
assert\_noop!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
false,\
), Error::::InvalidIndex);\
assert\_noop!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[12; 32].into()),\
0,\
false,\
), Error::::NoPermission);\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
false,\
));\
assert\_noop!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
true,\
), Error::::AlreadyConfirmed);\
})\
}

// list\_token function\
\#\[test]\
fn relist\_a\_nft() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(TokenListings::::get(1).unwrap().item\_id, 0);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 100);\
assert\_eq!(LocalAssetsFreezer::frozen\_balance(0, &\[1; 32].into()), Some(1));\
})\
}

\#\[test]\
fn relist\_nfts\_not\_created\_with\_marketplace\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(Nfts::create(\
RuntimeOrigin::signed(\[0; 32].into()),\
sp\_runtime::MultiAddress::Id(\[0; 32].into()),\
Default::default()\
));\
assert\_ok!(Nfts::mint(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
0,\
sp\_runtime::MultiAddress::Id(\[0; 32].into()),\
None\
));\
assert\_noop!(\
NftMarketplace::relist\_token(RuntimeOrigin::signed(\[0; 32].into()), 0, 0, 1000, 1),\
Error::::RegionUnknown\
);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_noop!(\
NftMarketplace::relist\_token(RuntimeOrigin::signed(\[0; 32].into()), 0, 0, 1000, 1),\
Error::::NftNotFound\
);\
})\
}

\#\[test]\
fn relist\_a\_nft\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_noop!(\
NftMarketplace::relist\_token(RuntimeOrigin::signed(\[0; 32].into()), 0, 0, 1000, 1),\
Error::::NotEnoughFunds\
);\
})\
}

// buy\_relisted\_token function\
\#\[test]\
fn buy\_relisted\_token\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_eq!(ForeignAssets::balance(1984, &(\[0; 32].into())), 20990000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 12000);\
assert\_eq!(ForeignAssets::balance(1984, &(\[1; 32].into())), 460\_000);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
3\
));\
assert\_ok!(NftMarketplace::buy\_relisted\_token(RuntimeOrigin::signed(\[3; 32].into()), 1, 2, crate::PaymentAssets::USDT));\
assert\_eq!(ForeignAssets::balance(1984, &(\[3; 32].into())), 3\_000);\
assert\_eq!(LocalAssets::balance(0, &\[3; 32].into()), 2);\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_ok!(NftMarketplace::buy\_relisted\_token(RuntimeOrigin::signed(\[3; 32].into()), 1, 1, crate::PaymentAssets::USDT));\
assert\_eq!(ForeignAssets::balance(1984, &(\[3; 32].into())), 2\_000);\
assert\_eq!(TokenListings::::get(1).is\_some(), false);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
500,\
1\
));\
assert\_ok!(NftMarketplace::buy\_relisted\_token(RuntimeOrigin::signed(\[3; 32].into()), 2, 1, crate::PaymentAssets::USDT));\
assert\_eq!(TokenListings::::get(0).is\_some(), false);\
assert\_eq!(PropertyOwner::::get(0).len(), 2);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[1; 32].into()), 96);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[3; 32].into()), 4);\
assert\_eq!(ForeignAssets::balance(1984, &(\[1; 32].into())), 463\_465);\
assert\_eq!(ForeignAssets::balance(1984, &(\[3; 32].into())), 1\_500);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 96);\
assert\_eq!(LocalAssets::balance(0, &\[3; 32].into()), 4);\
})\
}

\#\[test]\
fn buy\_relisted\_token\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_eq!(ForeignAssets::balance(1984, &(\[0; 32].into())), 20990000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 12\_000);\
assert\_eq!(ForeignAssets::balance(1984, &(\[1; 32].into())), 460\_000);\
assert\_eq!(RegisteredNftDetails::::get(0, 0).unwrap().spv\_created, true);\
assert\_noop!(\
NftMarketplace::buy\_relisted\_token(RuntimeOrigin::signed(\[3; 32].into()), 1, 1, crate::PaymentAssets::USDT),\
Error::::TokenNotForSale\
);\
})\
}

// make\_offer function\
\#\[test]\
fn make\_offer\_works() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
500,\
1\
));\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 2000, 1, crate::PaymentAssets::USDT));\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::property\_account\_id(0)), 0);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(2000));\
})\
}

\#\[test]\
fn make\_offer\_fails() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 200, 1, crate::PaymentAssets::USDT),\
Error::::TokenNotForSale\
);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
500,\
1\
));\
assert\_noop!(\
NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 200, 2, crate::PaymentAssets::USDT),\
Error::::NotEnoughTokenAvailable\
);\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 200, 1, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[3; 32].into()), 1, 300, 1, crate::PaymentAssets::USDT));\
assert\_noop!(\
NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 400, 1, crate::PaymentAssets::USDT),\
Error::::OnlyOneOfferPerUser\
);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).unwrap().token\_price, 200);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[3; 32].into()).unwrap().token\_price, 300);\
})\
}

// handle\_offer function\
\#\[test]\
fn handle\_offer\_works() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
5000,\
20\
));\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 200, 1, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[3; 32].into()), 1, 150, 1, crate::PaymentAssets::USDC));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(200));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1337, &\[3; 32].into()), Some(150));\
assert\_ok!(NftMarketplace::handle\_offer(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
\[2; 32].into(),\
crate::Offer::Reject\
));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), None);\
assert\_ok!(NftMarketplace::cancel\_offer(RuntimeOrigin::signed(\[3; 32].into()), 1));\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_none(), true);\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 2000, 10, crate::PaymentAssets::USDT));\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(20000));\
assert\_ok!(NftMarketplace::handle\_offer(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
\[2; 32].into(),\
crate::Offer::Accept\
));\
assert\_eq!(TokenListings::::get(1).unwrap().amount, 10);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_none(), true);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::property\_account\_id(1)), 0);\
assert\_eq!(LocalAssets::balance(0, &(\[1; 32].into())), 90);\
assert\_eq!(LocalAssets::balance(0, &(\[2; 32].into())), 10);\
assert\_eq!(LocalAssetsFreezer::frozen\_balance(0, &\[1; 32].into()), Some(10));\
assert\_eq!(ForeignAssets::balance(1984, &(\[1; 32].into())), 479\_800);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_130\_000);\
})\
}

\#\[test]\
fn handle\_offer\_fails() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
NftMarketplace::handle\_offer(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
\[2; 32].into(),\
crate::Offer::Reject\
),\
Error::::TokenNotForSale\
);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
5000,\
2\
));\
assert\_noop!(\
NftMarketplace::handle\_offer(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
\[2; 32].into(),\
crate::Offer::Reject\
),\
Error::::InvalidIndex\
);\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 200, 1, crate::PaymentAssets::USDT));\
assert\_noop!(\
NftMarketplace::handle\_offer(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
\[2; 32].into(),\
crate::Offer::Accept\
),\
Error::::NoPermission\
);\
})\
}

// cancel\_offer function\
\#\[test]\
fn cancel\_offer\_works() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
500,\
1\
));\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 2000, 1, crate::PaymentAssets::USDT));\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(2000));\
assert\_ok!(NftMarketplace::cancel\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1));\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_some(), false);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::property\_account\_id(1)), 0);\
})\
}

\#\[test]\
fn cancel\_offer\_fails() {\
new\_test\_ext().execute\_with(|| {\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
500,\
1\
));\
assert\_noop!(\
NftMarketplace::cancel\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1),\
Error::::InvalidIndex\
);\
assert\_ok!(NftMarketplace::make\_offer(RuntimeOrigin::signed(\[2; 32].into()), 1, 2000, 1, crate::PaymentAssets::USDT));\
assert\_eq!(TokenListings::::get(1).is\_some(), true);\
assert\_eq!(OngoingOffers::::get::\<u32, AccountId>(1, \[2; 32].into()).is\_some(), true);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 1\_150\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(2000));\
assert\_noop!(\
NftMarketplace::cancel\_offer(RuntimeOrigin::signed(\[1; 32].into()), 1),\
Error::::InvalidIndex\
);\
})\
}

// upgrade\_listing function\
\#\[test]\
fn upgrade\_price\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_ok!(NftMarketplace::upgrade\_listing(RuntimeOrigin::signed(\[1; 32].into()), 1, 300));\
assert\_eq!(TokenListings::::get(1).unwrap().token\_price, 300);\
})\
}

\#\[test]\
fn upgrade\_price\_fails\_if\_not\_owner() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_noop!(\
NftMarketplace::upgrade\_listing(RuntimeOrigin::signed(\[4; 32].into()), 1, 300),\
Error::::NoPermission\
);\
})\
}

// upgrade\_object function\
\#\[test]\
fn upgrade\_object\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::upgrade\_object(RuntimeOrigin::signed(\[0; 32].into()), 0, 30000));\
assert\_eq!(OngoingObjectListing::::get(0).unwrap().token\_price, 30000);\
})\
}

\#\[test]\
fn upgrade\_object\_and\_distribute\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 50, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::upgrade\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
20\_000\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 50, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_eq!(ForeignAssets::balance(1984, &(\[0; 32].into())), 21485000);\
assert\_eq!(ForeignAssets::balance(1984, \&NftMarketplace::treasury\_account\_id()), 22000);\
assert\_eq!(ForeignAssets::balance(1984, &(\[1; 32].into())), 980\_000);\
assert\_eq!(ForeignAssets::balance(1984, &(\[2; 32].into())), 110\_000);

```
	assert_eq!(RegisteredNftDetails::<Test>::get(0, 0).unwrap().spv_created, true);
	assert_eq!(ListedToken::<Test>::get(0), None);
})
```

}

\#\[test]\
fn upgrade\_single\_nft\_from\_listed\_object\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_noop!(\
NftMarketplace::upgrade\_listing(RuntimeOrigin::signed(\[0; 32].into()), 0, 300),\
Error::::TokenNotForSale\
);\
})\
}

\#\[test]\
fn upgrade\_object\_for\_relisted\_nft\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[0; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_noop!(\
NftMarketplace::upgrade\_object(RuntimeOrigin::signed(\[0; 32].into()), 1, 300),\
Error::::TokenNotForSale\
);\
})\
}

\#\[test]\
fn upgrade\_unknown\_collection\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_noop!(\
NftMarketplace::upgrade\_object(RuntimeOrigin::signed(\[0; 32].into()), 0, 300),\
Error::::TokenNotForSale\
);\
})\
}

// delist\_token function\
\#\[test]\
fn delist\_single\_token\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_ok!(NftMarketplace::delist\_token(RuntimeOrigin::signed(\[1; 32].into()), 1));\
assert\_eq!(TokenListings::::get(0), None);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
3\
));\
assert\_ok!(NftMarketplace::buy\_relisted\_token(RuntimeOrigin::signed(\[2; 32].into()), 2, 2, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::delist\_token(RuntimeOrigin::signed(\[1; 32].into()), 2));\
assert\_eq!(LocalAssets::balance(0, &\[2; 32].into()), 2);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 98);\
})\
}

\#\[test]\
fn delist\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
0,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
0,\
crate::LegalProperty::SpvSide,\
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
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
0,\
1000,\
1\
));\
assert\_noop!(\
NftMarketplace::delist\_token(RuntimeOrigin::signed(\[4; 32].into()), 1),\
Error::::NoPermission\
);\
assert\_noop!(\
NftMarketplace::delist\_token(RuntimeOrigin::signed(\[1; 32].into()), 2),\
Error::::TokenNotForSale\
);\
})\
}

\#\[test]\
fn listing\_objects\_in\_different\_regions() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 1, bvec!\[10, 10]));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 2, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
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
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
1,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
2,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 1, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
1,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
1,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
1,\
true,\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 2, 100, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[10; 32].into()),\
2,\
crate::LegalProperty::RealEstateDeveloperSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_claim\_property(\
RuntimeOrigin::signed(\[11; 32].into()),\
2,\
crate::LegalProperty::SpvSide,\
4\_000,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[10; 32].into()),\
2,\
true,\
));\
assert\_ok!(NftMarketplace::lawyer\_confirm\_documents(\
RuntimeOrigin::signed(\[11; 32].into()),\
2,\
true,\
));\
assert\_eq!(RegisteredNftDetails::::get(1, 0).unwrap().spv\_created, true);\
assert\_eq!(RegisteredNftDetails::::get(2, 0).unwrap().spv\_created, true);\
assert\_ok!(NftMarketplace::relist\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
1,\
0,\
1000,\
100\
));\
assert\_ok!(NftMarketplace::buy\_relisted\_token(\
RuntimeOrigin::signed(\[2; 32].into()),\
3,\
100,\
crate::PaymentAssets::USDT\
));\
assert\_eq!(LocalAssets::balance(1, &\[2; 32].into()), 100);\
assert\_eq!(LocalAssets::balance(2, &\[2; 32].into()), 100);\
})\
}

\#\[test]\
fn cancel\_buy\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_eq!(ListedToken::::get(0).unwrap(), 40);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 30);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[2; 32].into(), 0).token\_amount, 30);\
assert\_eq!(TokenBuyer::::get(0).len(), 2);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), Some(312\_000));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(312\_000));\
assert\_ok!(NftMarketplace::cancel\_buy(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(ListedToken::::get(0).unwrap(), 70);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), None);\
assert\_eq!(TokenBuyer::::get(0).len(), 1);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
})\
}

\#\[test]\
fn cancel\_buy\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_noop!(\
NftMarketplace::cancel\_buy(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::InvalidIndex\
);\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_noop!(\
NftMarketplace::cancel\_buy(RuntimeOrigin::signed(\[3; 32].into()), 0),\
Error::::NoTokenBought\
);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 40, crate::PaymentAssets::USDT));\
assert\_noop!(\
NftMarketplace::cancel\_buy(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::PropertyAlreadySold\
);\
})\
}

\#\[test]\
fn cancel\_buy\_fails\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 40, crate::PaymentAssets::USDT));\
run\_to\_block(40);\
assert\_noop!(\
NftMarketplace::cancel\_buy(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::ListingExpired\
);\
})\
}

\#\[test]\
fn refund\_expired\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_eq!(ListedToken::::get(0).unwrap(), 70);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 30);\
assert\_eq!(TokenBuyer::::get(0).len(), 1);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), Some(312\_000));\
run\_to\_block(40);\
assert\_ok!(NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(ListedToken::::get(0), None);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), None);\
})\
}

\#\[test]\
fn refund\_expired\_works\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
1\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 30, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[2; 32].into()), 0, 20, crate::PaymentAssets::USDT));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[3; 32].into()), 0, 4, crate::PaymentAssets::USDT));\
assert\_eq!(ListedToken::::get(0).unwrap(), 46);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 30);\
assert\_eq!(TokenBuyer::::get(0).len(), 3);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[2; 32].into()), 1\_150\_000);\
assert\_eq!(ForeignAssets::balance(1984, &\[3; 32].into()), 5\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), Some(31\_200));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[2; 32].into()), Some(20\_800));\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[3; 32].into()), Some(4\_160));\
run\_to\_block(40);\
assert\_ok!(NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[1; 32].into()), 0));\
assert\_eq!(ListedToken::::get(0), Some(76));\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[3; 32].into(), 0).token\_amount, 4);\
assert\_eq!(TokenBuyer::::get(0).len(), 2);\
assert\_eq!(ForeignAssets::balance(1984, &\[1; 32].into()), 1\_500\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[1; 32].into()), None);\
assert\_ok!(NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[2; 32].into()), 0));\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 10\_000);\
assert\_eq!(Balances::free\_balance(&(NftMarketplace::property\_account\_id(0))), 99);\
assert\_eq!(Balances::balance(&(NftMarketplace::property\_account\_id(0))), 99);\
assert\_ok!(NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[3; 32].into()), 0));\
assert\_eq!(Balances::free\_balance(&(NftMarketplace::property\_account\_id(0))), 0);\
assert\_eq!(Balances::balance(&(NftMarketplace::property\_account\_id(0))), 0);\
assert\_eq!(TokenOwner::::get::\<AccountId, u32>(\[1; 32].into(), 0).token\_amount, 0);\
assert\_eq!(TokenBuyer::::get(0).len(), 0);\
assert\_eq!(ForeignAssets::balance(1984, &\[3; 32].into()), 5\_000);\
assert\_eq!(ForeignAssetsFreezer::frozen\_balance(1984, &\[3; 32].into()), None);\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 0);\
})\
}

\#\[test]\
fn refund\_expired\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_noop!(\
NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::InvalidIndex\
);\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_noop!(\
NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::ListingNotExpired\
);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
run\_to\_block(40);\
assert\_noop!(\
NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[1; 32].into()), 0),\
Error::::PropertyAlreadySold\
);\
})\
}

\#\[test]\
fn refund\_expired\_fails\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 99, crate::PaymentAssets::USDT));\
run\_to\_block(40);\
assert\_noop!(\
NftMarketplace::refund\_expired(RuntimeOrigin::signed(\[2; 32].into()), 0),\
Error::::NoTokenBought\
);\
})\
}

\#\[test]\
fn send\_property\_token\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[3; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[1; 32].into()), 100);\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 100);\
assert\_eq!(PropertyOwner::::get(0).len(), 1);\
assert\_ok!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
\[2; 32].into(),\
20\
));\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 80);\
assert\_eq!(PropertyOwner::::get(0).len(), 2);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[1; 32].into()), 80);\
assert\_eq!(LocalAssets::balance(0, &\[2; 32].into()), 20);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[2; 32].into()), 20);\
assert\_ok!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
\[3; 32].into(),\
80\
));\
assert\_ok!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[2; 32].into()),\
0,\
\[3; 32].into(),\
20\
));\
assert\_eq!(LocalAssets::balance(0, &\[1; 32].into()), 0);\
assert\_eq!(PropertyOwner::::get(0).len(), 1);\
assert\_eq!(LocalAssets::balance(0, &\[2; 32].into()), 0);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[2; 32].into()), 0);\
assert\_eq!(LocalAssets::balance(0, &\[3; 32].into()), 100);\
assert\_eq!(PropertyOwnerToken::::get::\<u32, AccountId>(0, \[3; 32].into()), 100);\
})\
}

\#\[test]\
fn send\_property\_token\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_noop!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
\[2; 32].into(),\
20\
), Error::::UserNotWhitelisted);\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[1; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
assert\_noop!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[1; 32].into()),\
0,\
\[2; 32].into(),\
20\
), Error::::UserNotWhitelisted);\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[2; 32].into()));\
assert\_noop!(NftMarketplace::send\_property\_token(\
RuntimeOrigin::signed(\[2; 32].into()),\
1,\
\[1; 32].into(),\
20\
), Error::::NotEnoughToken);\
})\
}

\#\[test]\
fn reclaim\_unsold\_works() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
assert\_ok!(NftMarketplace::create\_new\_location(RuntimeOrigin::root(), 0, bvec!\[10, 10]));\
assert\_ok!(XcavateWhitelist::add\_to\_whitelist(RuntimeOrigin::root(), \[0; 32].into()));\
assert\_ok!(NftMarketplace::list\_object(\
RuntimeOrigin::signed(\[0; 32].into()),\
0,\
bvec!\[10, 10],\
10\_000,\
100,\
bvec!\[22, 22]\
));\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 100\_000);\
run\_to\_block(40);\
assert\_ok!(NftMarketplace::reclaim\_unsold(RuntimeOrigin::signed(\[0; 32].into()), 0));\
assert\_eq!(ListedToken::::get(0), None);\
assert\_eq!(pallet\_nfts::Item::::get(0, 0).is\_none(), true);\
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 0);\
})\
}

\#\[test]\
fn reclaim\_unsold\_fails() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 100\_000);\
run\_to\_block(20);\
assert\_noop!(NftMarketplace::reclaim\_unsold(RuntimeOrigin::signed(\[0; 32].into()), 0), Error::::ListingNotExpired);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 10, crate::PaymentAssets::USDT));\
assert\_noop!(NftMarketplace::reclaim\_unsold(RuntimeOrigin::signed(\[0; 32].into()), 1), Error::::InvalidIndex);\
run\_to\_block(40);\
assert\_noop!(NftMarketplace::reclaim\_unsold(RuntimeOrigin::signed(\[0; 32].into()), 0), Error::::TokenNotReturned);\
})\
}

\#\[test]\
fn reclaim\_unsold\_fails\_2() {\
new\_test\_ext().execute\_with(|| {\
System::set\_block\_number(1);\
assert\_ok!(NftMarketplace::create\_new\_region(RuntimeOrigin::signed(\[8; 32].into()), 30));\
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
assert\_eq!(Balances::balance\_on\_hold(\&HoldReason::ListingDepositReserve.into(), &(\[0; 32].into())), 100\_000);\
run\_to\_block(20);\
assert\_ok!(NftMarketplace::buy\_token(RuntimeOrigin::signed(\[1; 32].into()), 0, 100, crate::PaymentAssets::USDT));\
run\_to\_block(40);\
assert\_noop!(NftMarketplace::reclaim\_unsold(RuntimeOrigin::signed(\[0; 32].into()), 0), Error::::PropertyAlreadySold);\
})\
}
