# Online real output

## SHOW ENGINE INNODB STATUS

```log
=====================================
2022-10-18 15:46:23 0x3994 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 10 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 518126 srv_active, 0 srv_shutdown, 2409362 srv_idle
srv_master_thread log flush and writes: 0
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 7269063
OS WAIT ARRAY INFO: signal count 20572173
RW-shared spins 0, rounds 0, OS waits 0
RW-excl spins 0, rounds 0, OS waits 0
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 0.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
------------------------
LATEST DETECTED DEADLOCK
------------------------
2022-10-13 17:32:33 0x9e0
*** (1) TRANSACTION:
TRANSACTION 25774544, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1128, 2 row lock(s), undo log entries 1
MySQL thread id 82089, OS thread handle 14792, query id 467582450 localhost 127.0.0.1 ecnt updating
update lab_labour set
name = '李宇航',
idcard = null,
mobile = null,
sex_id = null,
birth_date = null,
education_id = null,
major = null,
graduate_school = null,
email = null,
idcard_date=null,
province = 'P0i2zTBOwJ6wWLmxkNMuSg==',
city = 'Lny+eg7w5eD4rJQCO7G4Gw==',
country = 'bAKJKcoo6uxQyygQqqtyuQ==',
street = 'I4yXMmxZ+2wXIoCrd+TAug==',
postal_code = null,
household_province = 'P0i2zTBOwJ6wWLmxkNMuSg==',
household_city = 'Lny+eg7w5eD4rJQCO7G4Gw==',
household_country = 'sMt2RAOAg0OYK3tPD/6ANg==',
household_street = 'EKvhUz5m5+TSu12rs/6GSRF2kYRkkXLDrPjKU9uVcNY=',
expected_salary = 3500.0,
working_life = 3,
work_province = 'P0i2zTBOwJ6wWLmxkNMuSg==',
work_city = 'Lny+eg7w5eD4rJQCO7G4Gw==',
role_id = 3479,
entry_date = 99,
user_id =54179
where id = 14998

*** (1) HOLDS THE LOCK(S):
RECORD LOCKS space id 1826 page no 266 n bits 80 index PRIMARY of table `ecnt`.`sys_user` trx id 25774544 lock_mode X locks rec but not gap
Record lock, heap no 13 PHYSICAL RECORD: n_fields 29; compact format; info bits 64
 0: len 4; hex 8000d3a3; asc     ;;
 1: len 6; hex 0000018949d0; asc     I ;;
 2: len 7; hex 81000001bf0110; asc        ;;
 3: len 11; hex 3139393137373239393039; asc 19917729909;;
 4: SQL NULL;
 5: len 9; hex e69d8ee5ae87e888aa; asc          ;;
 6: SQL NULL;
 7: len 4; hex 80000000; asc     ;;
 8: len 4; hex 80000000; asc     ;;
 9: len 4; hex 80000000; asc     ;;
 10: SQL NULL;
 11: len 4; hex 80000003; asc     ;;
 12: len 11; hex 3139393137373239393039; asc 19917729909;;
 13: SQL NULL;
 14: len 17; hex 313938363430313731314071712e636f6d; asc 1986401711@qq.com;;
 15: SQL NULL;
 16: len 4; hex 8000001f; asc     ;;
 17: len 30; hex 457a454d334e73685337316c6966686c4f4c68794a6b6b42354848786843; asc EzEM3NshS71lifhlOLhyJkkB5HHxhC; (total 44 bytes);
 18: len 30; hex 36446d6e36764b6354346a7866534571305a58685a765843795773613136; asc 6Dmn6vKcT4jxfSEq0ZXhZvXCyWsa16; (total 44 bytes);
 19: len 30; hex 336154693335384c727a477433752b65746f5158622f4853674434384956; asc 3aTi358LrzGt3u+etoQXb/HSgD48IV; (total 44 bytes);
 20: len 3; hex 8f9c48; asc   H;;
 21: len 5; hex 99ae1b1821; asc     !;;
 22: len 4; hex 7fffffff; asc     ;;
 23: len 4; hex 80000002; asc     ;;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: len 5; hex 99be4c0000; asc   L  ;;


*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 4830 page no 502 n bits 96 index PRIMARY of table `ecnt`.`lab_labour` trx id 25774544 lock_mode X locks rec but not gap waiting
Record lock, heap no 15 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a96; asc   : ;;
 1: len 6; hex 0000018900c2; asc       ;;
 2: len 7; hex 01000000851792; asc        ;;
 3: len 28; hex 6f546e4469354a724a4b416e7a372d7731374937546c7177676b7834; asc oTnDi5JrJKAnz7-w17I7Tlqwgkx4;;
 4: len 9; hex e69d8ee5ae87e888aa; asc          ;;
 5: len 30; hex 336154693335384c727a477433752b65746f5158622f4853674434384956; asc 3aTi358LrzGt3u+etoQXb/HSgD48IV; (total 44 bytes);
 6: len 4; hex 80000003; asc     ;;
 7: len 3; hex 8f9c48; asc   H;;
 8: len 4; hex 8000001f; asc     ;;
 9: len 30; hex 457a454d334e73685337316c6966686c4f4c68794a6b6b42354848786843; asc EzEM3NshS71lifhlOLhyJkkB5HHxhC; (total 44 bytes);
 10: len 30; hex 36446d6e36764b6354346a7866534571305a58685a765843795773613136; asc 6Dmn6vKcT4jxfSEq0ZXhZvXCyWsa16; (total 44 bytes);
 11: len 11; hex 3139393137373239393039; asc 19917729909;;
 12: len 17; hex 313938363430313731314071712e636f6d; asc 1986401711@qq.com;;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 62414b4a4b636f6f36757851797967517171747975513d3d; asc bAKJKcoo6uxQyygQqqtyuQ==;;
 16: len 24; hex 493479584d6d785a2b327758496f4372642b544175673d3d; asc I4yXMmxZ+2wXIoCrd+TAug==;;
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 734d743252414f4167304f594b337450442f36414e673d3d; asc sMt2RAOAg0OYK3tPD/6ANg==;;
 21: len 30; hex 454b7668557a356d352b545375313272732f3647535246326b59526b6b58; asc EKvhUz5m5+TSu12rs/6GSRF2kYRkkX; (total 44 bytes);
 22: len 9; hex 8000000000000dac00; asc          ;;
 23: len 4; hex 80000003; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 80000063; asc    c;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0a97; asc      ;;
 30: len 4; hex 80000001; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f3; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: len 5; hex 99be4c0000; asc   L  ;;
 45: SQL NULL;


*** (2) TRANSACTION:
TRANSACTION 25774543, ACTIVE 0 sec fetching rows
mysql tables in use 1, locked 1
LOCK WAIT 560 lock struct(s), heap size 73848, 21360 row lock(s)
MySQL thread id 81790, OS thread handle 15748, query id 467582446 localhost 127.0.0.1 ecnt updating
update sys_user
set mobile = null
where
mobile
= '18893147674'

*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 4830 page no 502 n bits 96 index PRIMARY of table `ecnt`.`lab_labour` trx id 25774543 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 9 PHYSICAL RECORD: n_fields 46; compact format; info bits 64
 0: len 4; hex 80003a97; asc   : ;;
 1: len 6; hex 00000188eef8; asc       ;;
 2: len 7; hex 81000000a8012a; asc       *;;
 3: len 28; hex 6f546e446935506e3659426145527055475773773662303955346673; asc oTnDi5Pn6YBaERpUGWsw6b09U4fs;;
 4: len 6; hex e78e8be5ae89; asc       ;;
 5: SQL NULL;
 6: len 4; hex 80000002; asc     ;;
 7: SQL NULL;
 8: len 4; hex 8000001d; asc     ;;
 9: len 24; hex 4f3842686e757632556d4f522b4a6c58346343466e673d3d; asc O8Bhnuv2UmOR+JlX4cCFng==;;
 10: SQL NULL;
 11: len 11; hex 3138323034363635333735; asc 18204665375;;
 12: len 16; hex 3738313630383631354071712e636f6d; asc 781608615@qq.com;;
 13: SQL NULL;
 14: SQL NULL;
 15: SQL NULL;
 16: SQL NULL;
 17: SQL NULL;
 18: SQL NULL;
 19: SQL NULL;
 20: SQL NULL;
 21: SQL NULL;
 22: SQL NULL;
 23: SQL NULL;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: SQL NULL;
 29: len 5; hex 99ae1b0b62; asc     b;;
 30: len 4; hex 80000000; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f6; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 10 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a94; asc   : ;;
 1: len 6; hex 00000188f19f; asc       ;;
 2: len 7; hex 02000001252e16; asc     %. ;;
 3: len 28; hex 6f546e446935444a48525232596a3357396169374e4a767939524177; asc oTnDi5DJHRR2Yj3W9ai7NJvy9RAw;;
 4: len 6; hex e4b990e59bad; asc       ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 45356352382b4d7750344c2f467634675a64555257673d3d; asc E5cR8+MwP4L/Fv4gZdURWg==;;
 14: len 24; hex 3348745a4a4e626134615a586c7754515377565149773d3d; asc 3HtZJNba4aZXlwTQSwVQIw==;;
 15: len 24; hex 41744c523737737a634454564b42772b6465336f4f773d3d; asc AtLR77szcDTVKBw+de3oOw==;;
 16: len 30; hex 2f7945443351474733324c576d70346267497154596d5234594a414e6574; asc /yED3QGG32LWmp4bgIqTYmR4YJANet; (total 44 bytes);
 17: SQL NULL;
 18: len 24; hex 45356352382b4d7750344c2f467634675a64555257673d3d; asc E5cR8+MwP4L/Fv4gZdURWg==;;
 19: len 24; hex 3348745a4a4e626134615a586c7754515377565149773d3d; asc 3HtZJNba4aZXlwTQSwVQIw==;;
 20: len 24; hex 41744c523737737a634454564b42772b6465336f4f773d3d; asc AtLR77szcDTVKBw+de3oOw==;;
 21: len 30; hex 2f7945443351474733324c576d70346267497154596d5234594a414e6574; asc /yED3QGG32LWmp4bgIqTYmR4YJANet; (total 44 bytes);
 22: len 9; hex 800000000000271000; asc       '  ;;
 23: len 4; hex 8000000e; asc     ;;
 24: len 24; hex 45356352382b4d7750344c2f467634675a64555257673d3d; asc E5cR8+MwP4L/Fv4gZdURWg==;;
 25: len 24; hex 3348745a4a4e626134615a586c7754515377565149773d3d; asc 3HtZJNba4aZXlwTQSwVQIw==;;
 26: len 4; hex 80000015; asc     ;;
 27: len 4; hex 8000000a; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0a56; asc     V;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f1; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3731343530; asc 71450;;
 42: len 4; hex 8000000e; asc     ;;
 43: len 4; hex 8000d394; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 11 PHYSICAL RECORD: n_fields 46; compact format; info bits 64
 0: len 4; hex 80003a98; asc   : ;;
 1: len 6; hex 00000188f04b; asc      K;;
 2: len 7; hex 8200000087012a; asc       *;;
 3: len 28; hex 6f546e446935426569553075787a324e7044447730786d516931776b; asc oTnDi5BeiU0uxz2NpDDw0xmQi1wk;;
 4: len 9; hex e69d8ee59889e6aca3; asc          ;;
 5: SQL NULL;
 6: len 4; hex 80000003; asc     ;;
 7: SQL NULL;
 8: len 4; hex 8000001a; asc     ;;
 9: len 30; hex 41724f44616d6e526a7447786542346f4b5834714e6a472b4852775a6548; asc ArODamnRjtGxeB4oKX4qNjG+HRwZeH; (total 44 bytes);
 10: SQL NULL;
 11: len 11; hex 3133303739363233313939; asc 13079623199;;
 12: len 16; hex 3833393930393838354071712e636f6d; asc 839909885@qq.com;;
 13: SQL NULL;
 14: SQL NULL;
 15: SQL NULL;
 16: SQL NULL;
 17: SQL NULL;
 18: SQL NULL;
 19: SQL NULL;
 20: SQL NULL;
 21: SQL NULL;
 22: SQL NULL;
 23: SQL NULL;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: SQL NULL;
 29: len 5; hex 99ae1b0b8b; asc      ;;
 30: len 4; hex 80000000; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f7; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 12 PHYSICAL RECORD: n_fields 46; compact format; info bits 64
 0: len 4; hex 80003a99; asc   : ;;
 1: len 6; hex 00000188f1a7; asc       ;;
 2: len 7; hex 81000000d3012a; asc       *;;
 3: len 28; hex 6f546e446935446d4f5a467a5f3953574e4e634a4b396b4761694449; asc oTnDi5DmOZFz_9SWNNcJK9kGaiDI;;
 4: len 6; hex e999b6e5ae87; asc       ;;
 5: SQL NULL;
 6: len 4; hex 80000002; asc     ;;
 7: SQL NULL;
 8: len 4; hex 8000001f; asc     ;;
 9: len 30; hex 767873754e2b4a37642f5930454f447677593141476368514a4552487839; asc vxsuN+J7d/Y0EODvwY1AGchQJERHx9; (total 44 bytes);
 10: SQL NULL;
 11: len 11; hex 3133333239353938393839; asc 13329598989;;
 12: len 17; hex 313332303431363832304071712e636f6d; asc 1320416820@qq.com;;
 13: SQL NULL;
 14: SQL NULL;
 15: SQL NULL;
 16: SQL NULL;
 17: SQL NULL;
 18: SQL NULL;
 19: SQL NULL;
 20: SQL NULL;
 21: SQL NULL;
 22: SQL NULL;
 23: SQL NULL;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: SQL NULL;
 29: len 5; hex 99ae1b0bc2; asc      ;;
 30: len 4; hex 80000000; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f8; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 14 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a92; asc   : ;;
 1: len 6; hex 00000188fbd6; asc       ;;
 2: len 7; hex 010000011d1b31; asc       1;;
 3: len 28; hex 6f546e446935413175775431344d5039614c724951766f4556474534; asc oTnDi5A1uwT14MP9aLrIQvoEVGE4;;
 4: len 6; hex e6bb95e7a38a; asc       ;;
 5: len 30; hex 454c344b6a384d703443356d45612f54574c6578656159305a31754d2f6a; asc EL4Kj8Mp4C5mEa/TWLexeaY0Z1uM/j; (total 44 bytes);
 6: len 4; hex 80000002; asc     ;;
 7: len 3; hex 8f8ac8; asc    ;;
 8: len 4; hex 8000001d; asc     ;;
 9: len 24; hex 5a316a4463455071664544475541444c5138334276413d3d; asc Z1jDcEPqfEDGUADLQ83BvA==;;
 10: len 24; hex 36596f4e796647704474474977397a784a4c2f7341513d3d; asc 6YoNyfGpDtGIw9zxJL/sAQ==;;
 11: len 11; hex 3138353034353231313131; asc 18504521111;;
 12: len 14; hex 313437363236324071712e636f6d; asc 1476262@qq.com;;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 16: len 30; hex 2f324a2b5033624c574a38666378536339736d774b6e764e4f74515a4d69; asc /2J+P3bLWJ8fcxSc9smwKnvNOtQZMi; (total 44 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 21: len 30; hex 5a616f64673056516549334a6d4939715046422b51563161766953436453; asc Zaodg0VQeI3JmI9qPFB+QV1aviSCdS; (total 44 bytes);
 22: len 9; hex 8000000000000dac00; asc          ;;
 23: len 4; hex 80000004; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 80000063; asc    c;;
 28: SQL NULL;
 29: len 5; hex 99ae1b09c8; asc      ;;
 30: len 4; hex 80000001; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126ee; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: len 5; hex 99ddd20000; asc      ;;
 45: SQL NULL;

Record lock, heap no 15 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a96; asc   : ;;
 1: len 6; hex 0000018900c2; asc       ;;
 2: len 7; hex 01000000851792; asc        ;;
 3: len 28; hex 6f546e4469354a724a4b416e7a372d7731374937546c7177676b7834; asc oTnDi5JrJKAnz7-w17I7Tlqwgkx4;;
 4: len 9; hex e69d8ee5ae87e888aa; asc          ;;
 5: len 30; hex 336154693335384c727a477433752b65746f5158622f4853674434384956; asc 3aTi358LrzGt3u+etoQXb/HSgD48IV; (total 44 bytes);
 6: len 4; hex 80000003; asc     ;;
 7: len 3; hex 8f9c48; asc   H;;
 8: len 4; hex 8000001f; asc     ;;
 9: len 30; hex 457a454d334e73685337316c6966686c4f4c68794a6b6b42354848786843; asc EzEM3NshS71lifhlOLhyJkkB5HHxhC; (total 44 bytes);
 10: len 30; hex 36446d6e36764b6354346a7866534571305a58685a765843795773613136; asc 6Dmn6vKcT4jxfSEq0ZXhZvXCyWsa16; (total 44 bytes);
 11: len 11; hex 3139393137373239393039; asc 19917729909;;
 12: len 17; hex 313938363430313731314071712e636f6d; asc 1986401711@qq.com;;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 62414b4a4b636f6f36757851797967517171747975513d3d; asc bAKJKcoo6uxQyygQqqtyuQ==;;
 16: len 24; hex 493479584d6d785a2b327758496f4372642b544175673d3d; asc I4yXMmxZ+2wXIoCrd+TAug==;;
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 734d743252414f4167304f594b337450442f36414e673d3d; asc sMt2RAOAg0OYK3tPD/6ANg==;;
 21: len 30; hex 454b7668557a356d352b545375313272732f3647535246326b59526b6b58; asc EKvhUz5m5+TSu12rs/6GSRF2kYRkkX; (total 44 bytes);
 22: len 9; hex 8000000000000dac00; asc          ;;
 23: len 4; hex 80000003; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 80000063; asc    c;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0a97; asc      ;;
 30: len 4; hex 80000001; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f3; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: len 5; hex 99be4c0000; asc   L  ;;
 45: SQL NULL;

Record lock, heap no 16 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a91; asc   : ;;
 1: len 6; hex 00000189492e; asc     I.;;
 2: len 7; hex 02000000f215c1; asc        ;;
 3: len 28; hex 6f546e44693545636b6944685237516d78375f566a4a51523261724d; asc oTnDi5EckiDhR7Qmx7_VjJQR2arM;;
 4: len 6; hex e5a79ce8bfaa; asc       ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 16: len 30; hex 47764f454d78795569733274386133376564665936384e314f6f43724379; asc GvOEMxyUis2t8a37edfY68N1OoCrCy; (total 44 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 21: len 30; hex 47764f454d78795569733274386133376564665936384e314f6f43724379; asc GvOEMxyUis2t8a37edfY68N1OoCrCy; (total 44 bytes);
 22: len 9; hex 8000000000000fa000; asc          ;;
 23: len 4; hex 8000000d; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d89; asc     ;;
 27: len 4; hex 80000018; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b099d; asc      ;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126ed; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d3a2; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 17 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a90; asc   : ;;
 1: len 6; hex 0000018947a4; asc     G ;;
 2: len 7; hex 020000018b22ca; asc      " ;;
 3: len 28; hex 6f546e446935487a756a33466352476e726b502d7a716f3642684a55; asc oTnDi5Hzuj3FcRGnrkP-zqo6BhJU;;
 4: len 9; hex e78e8be889b3e698ad; asc          ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 15: len 24; hex 4e4c6e354f536e425671582b567a79564d52455465773d3d; asc NLn5OSnBVqX+VzyVMRETew==;;
 16: len 30; hex 795078344d357a2f444f384b7542756650423457656c665077766a516142; asc yPx4M5z/DO8KuBufPB4WelfPwvjQaB; (total 64 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 20: len 24; hex 4e4c6e354f536e425671582b567a79564d52455465773d3d; asc NLn5OSnBVqX+VzyVMRETew==;;
 21: len 30; hex 784642756d7657364e714e33324269632b4c74636e3034657056356d374d; asc xFBumvW6NqN32Bic+Ltcn04epV5m7M; (total 44 bytes);
 22: len 9; hex 8000000000000dac00; asc          ;;
 23: len 4; hex 80000010; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 80000063; asc    c;;
 28: SQL NULL;
 29: len 5; hex 99ae1b084e; asc     N;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126d9; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d3a0; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 18 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9c; asc   : ;;
 1: len 6; hex 000001892d7a; asc     -z;;
 2: len 7; hex 01000005d30420; asc        ;;
 3: SQL NULL;
 4: len 9; hex e8b0ade5ae97e6b38a; asc          ;;
 5: SQL NULL;
 6: len 4; hex 80000002; asc     ;;
 7: SQL NULL;
 8: len 4; hex 8000001c; asc     ;;
 9: len 24; hex 774d6232364656564b6b6f6c666c76566d3662617a413d3d; asc wMb26FVVKkolflvVm6bazA==;;
 10: SQL NULL;
 11: len 11; hex 3138303734333939353237; asc 18074399527;;
 12: len 18; hex 3138303734333939353237403138392e636e; asc 18074399527@189.cn;;
 13: SQL NULL;
 14: SQL NULL;
 15: SQL NULL;
 16: SQL NULL;
 17: SQL NULL;
 18: SQL NULL;
 19: SQL NULL;
 20: SQL NULL;
 21: SQL NULL;
 22: SQL NULL;
 23: SQL NULL;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: SQL NULL;
 29: len 5; hex 99ae1b1178; asc     x;;
 30: len 4; hex 80000000; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 80012780; asc   ' ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3731343232; asc 71422;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 19 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9b; asc   : ;;
 1: len 6; hex 000001891e1c; asc       ;;
 2: len 7; hex 010000013e2568; asc     >%h;;
 3: len 28; hex 6f546e4469354f596d714c56417346306a77765a33374f6949466759; asc oTnDi5OYmqLVAsF0jwvZ37OiIFgY;;
 4: len 9; hex e4b8a5e6a59ae7809a; asc          ;;
 5: len 30; hex 753148666879626b337658514e664547344d6b4765646153564962463768; asc u1Hfhybk3vXQNfEG4MkGedaSVIbF7h; (total 44 bytes);
 6: len 4; hex 80000002; asc     ;;
 7: len 3; hex 8f96e5; asc    ;;
 8: len 4; hex 8000001b; asc     ;;
 9: len 24; hex 7a385653686c75423873306331462f57557847596c513d3d; asc z8VShluB8s0c1F/WUxGYlQ==;;
 10: len 30; hex 6e36517444336f2b612f484d426732346d3444634636313669612f2b4353; asc n6QtD3o+a/HMBg24m4DcF616ia/+CS; (total 44 bytes);
 11: len 11; hex 3133333136333439383736; asc 13316349876;;
 12: len 16; hex 3635373630373832384071712e636f6d; asc 657607828@qq.com;;
 13: len 24; hex 316548534c654f43532f446c34726a535162736d6e513d3d; asc 1eHSLeOCS/Dl4rjSQbsmnQ==;;
 14: len 24; hex 43794c3959677659717a4432524a382f4b477a6c6c673d3d; asc CyL9YgvYqzD2RJ8/KGzllg==;;
 15: len 24; hex 5752555332583379366842333648384171454a4a51773d3d; asc WRUS2X3y6hB36H8AqEJJQw==;;
 16: len 24; hex 46754a65466441464a424152495066784e77745635413d3d; asc FuJeFdAFJBARIPfxNwtV5A==;;
 17: SQL NULL;
 18: len 24; hex 316548534c654f43532f446c34726a535162736d6e513d3d; asc 1eHSLeOCS/Dl4rjSQbsmnQ==;;
 19: len 24; hex 43794c3959677659717a4432524a382f4b477a6c6c673d3d; asc CyL9YgvYqzD2RJ8/KGzllg==;;
 20: len 24; hex 5752555332583379366842333648384171454a4a51773d3d; asc WRUS2X3y6hB36H8AqEJJQw==;;
 21: len 24; hex 46754a65466441464a424152495066784e77745635413d3d; asc FuJeFdAFJBARIPfxNwtV5A==;;
 22: len 9; hex 800000000000138800; asc          ;;
 23: len 4; hex 80000004; asc     ;;
 24: len 24; hex 316548534c654f43532f446c34726a535162736d6e513d3d; asc 1eHSLeOCS/Dl4rjSQbsmnQ==;;
 25: len 24; hex 43794c3959677659717a4432524a382f4b477a6c6c673d3d; asc CyL9YgvYqzD2RJ8/KGzllg==;;
 26: len 4; hex 80000d72; asc    r;;
 27: len 4; hex 8000002c; asc    ,;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0dc6; asc      ;;
 30: len 4; hex 80000001; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126fe; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3731333933; asc 71393;;
 42: len 4; hex 8000000f; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: len 5; hex 99b6280000; asc   (  ;;
 45: SQL NULL;

Record lock, heap no 20 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9a; asc   : ;;
 1: len 6; hex 0000018945f3; asc     E ;;
 2: len 7; hex 010000009e20fd; asc        ;;
 3: len 28; hex 6f546e4469354254505268496f5765624356765964785f7a49526949; asc oTnDi5BTPRhIoWebCVvYdx_zIRiI;;
 4: len 6; hex e78e8be6958f; asc       ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 6c6f4547546d4b38513363384933715a537152694f673d3d; asc loEGTmK8Q3c8I3qZSqRiOg==;;
 16: len 30; hex 31513033534c645a3065484a57346d2b756f42446a6e79632b486c74686a; asc 1Q03SLdZ0eHJW4m+uoBDjnyc+Hlthj; (total 44 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 6c6f4547546d4b38513363384933715a537152694f673d3d; asc loEGTmK8Q3c8I3qZSqRiOg==;;
 21: len 24; hex 6c6f4547546d4b38513363384933715a537152694f673d3d; asc loEGTmK8Q3c8I3qZSqRiOg==;;
 22: len 9; hex 8000000000000d0500; asc          ;;
 23: len 4; hex 80000004; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 80000064; asc    d;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0c24; asc     $;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f9; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d39d; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 21 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a95; asc   : ;;
 1: len 6; hex 000001894736; asc     G6;;
 2: len 7; hex 010000016e1c2f; asc     n /;;
 3: len 28; hex 6f546e4469354b766642714c4f367154764461336c4c4149707a7745; asc oTnDi5KvfBqLO6qTvDa3lLAIpzwE;;
 4: len 6; hex e5bca0e685a7; asc       ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 6c6f4547546d4b38513363384933715a537152694f673d3d; asc loEGTmK8Q3c8I3qZSqRiOg==;;
 16: len 30; hex 3839617657766d4c457651356370524a65366a707a4962484f4b71792f58; asc 89avWvmLEvQ5cpRJe6jpzIbHOKqy/X; (total 90 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 6c6f4547546d4b38513363384933715a537152694f673d3d; asc loEGTmK8Q3c8I3qZSqRiOg==;;
 21: len 30; hex 5a69664e566637416654306376684a52696f656e4f635a6a414766357959; asc ZifNVf7AfT0cvhJRioenOcZjAGf5yY; (total 44 bytes);
 22: len 9; hex 8000000000000bb800; asc          ;;
 23: len 4; hex 8000000f; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000da4; asc     ;;
 27: len 4; hex 80000063; asc    c;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0a6e; asc     n;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126f2; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d39f; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 22 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a8f; asc   : ;;
 1: len 6; hex 000001894670; asc     Fp;;
 2: len 7; hex 02000001b11b2f; asc       /;;
 3: len 28; hex 6f546e44693543655350695044584d787332362d2d6f754746466a41; asc oTnDi5CeSPiPDXMxs26--ouGFFjA;;
 4: len 6; hex e78e8be4baae; asc       ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 15: len 30; hex 57754f72377468635366355a486c6f424e62364d542f6b327a7358395947; asc WuOr7thcSf5ZHloBNb6MT/k2zsX9YG; (total 44 bytes);
 16: len 30; hex 57754f72377468635366355a486c6f424e62364d54373270686d31525a37; asc WuOr7thcSf5ZHloBNb6MT72phm1RZ7; (total 110 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 20: len 30; hex 57754f72377468635366355a486c6f424e62364d542f6b327a7358395947; asc WuOr7thcSf5ZHloBNb6MT/k2zsX9YG; (total 44 bytes);
 21: len 30; hex 7173466f7851572f307143412b386855636d415a68764b7959496d67624b; asc qsFoxQW/0qCA+8hUcmAZhvKyYImgbK; (total 130 bytes);
 22: len 9; hex 8000000000000e7400; asc        t ;;
 23: len 4; hex 8000000d; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4b5642466e364f4d7a4667522b4a54443675495134513d3d; asc KVBFn6OMzFgR+JTD6uIQ4Q==;;
 26: len 4; hex 80000d97; asc     ;;
 27: len 4; hex 8000000a; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b07a9; asc      ;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126d8; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d39e; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 25 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9e; asc   : ;;
 1: len 6; hex 0000018936da; asc     6 ;;
 2: len 7; hex 010000013b1da0; asc     ;  ;;
 3: len 28; hex 6f546e446935426c613947775a384e53673244594d31636635333134; asc oTnDi5Bla9GwZ8NSg2DYM1cf5314;;
 4: len 6; hex e7868ae5a587; asc       ;;
 5: len 30; hex 77724741626f67736f5a6746315a474a7a6a4743324d6a4d4972664e467a; asc wrGAbogsoZgF1ZGJzjGC2MjMIrfNFz; (total 44 bytes);
 6: len 4; hex 80000002; asc     ;;
 7: len 3; hex 8f8b4e; asc   N;;
 8: len 4; hex 8000001a; asc     ;;
 9: len 24; hex 6d5a502b456531524f526a5958373375716246712b673d3d; asc mZP+Ee1RORjYX73uqbFq+g==;;
 10: len 30; hex 36652f4f472f346f4674636a46734f4f6e4f3173686358366f5251747249; asc 6e/OG/4oFtcjFsOOnO1shcX6oRQtrI; (total 44 bytes);
 11: len 11; hex 3135393237383334323234; asc 15927834224;;
 12: len 16; hex 3731383438383232354071712e636f6d; asc 718488225@qq.com;;
 13: len 24; hex 4d4676682f54796167743561397a707959656c3264673d3d; asc MFvh/Tyagt5a9zpyYel2dg==;;
 14: len 24; hex 5146434e5664376a70634153647933577150764f33673d3d; asc QFCNVd7jpcASdy3WqPvO3g==;;
 15: len 24; hex 63754639416f4a4368757a586f4e4847516b647466673d3d; asc cuF9AoJChuzXoNHGQkdtfg==;;
 16: len 30; hex 546f636364752b3349667861747a6541566871396246364644697a4f4752; asc Toccdu+3IfxatzeAVhq9bF6FDizOGR; (total 64 bytes);
 17: SQL NULL;
 18: len 24; hex 4d4676682f54796167743561397a707959656c3264673d3d; asc MFvh/Tyagt5a9zpyYel2dg==;;
 19: len 24; hex 5146434e5664376a70634153647933577150764f33673d3d; asc QFCNVd7jpcASdy3WqPvO3g==;;
 20: len 24; hex 63754639416f4a4368757a586f4e4847516b647466673d3d; asc cuF9AoJChuzXoNHGQkdtfg==;;
 21: len 30; hex 546f636364752b3349667861747a6541566871396246364644697a4f4752; asc Toccdu+3IfxatzeAVhq9bF6FDizOGR; (total 64 bytes);
 22: len 9; hex 8000000000001e7800; asc        x ;;
 23: len 4; hex 8000000b; asc     ;;
 24: len 24; hex 4d4676682f54796167743561397a707959656c3264673d3d; asc MFvh/Tyagt5a9zpyYel2dg==;;
 25: len 30; hex 6a554e54704276417176744854314d617674614555506236784f54495176; asc jUNTpBvAqvtHT1MavtaEUPb6xOTIQv; (total 44 bytes);
 26: len 4; hex 80000015; asc     ;;
 27: len 4; hex 80000007; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b13e6; asc      ;;
 30: len 4; hex 80000001; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 80012787; asc   ' ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3731343232; asc 71422;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 80000000; asc     ;;
 44: len 5; hex 99edda0000; asc      ;;
 45: SQL NULL;

Record lock, heap no 27 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a93; asc   : ;;
 1: len 6; hex 0000018943c8; asc     C ;;
 2: len 7; hex 02000000ee159c; asc        ;;
 3: len 28; hex 6f546e446935436c5257332d7a5a4f7033487771334e32566551556f; asc oTnDi5ClRW3-zZOp3Hwq3N2VeQUo;;
 4: len 9; hex e688bfe7baa2e7bbb4; asc          ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 16: len 30; hex 50333135637173794a6f57717742483568646d73544d57416d5541413257; asc P315cqsyJoWqwBH5hdmsTMWAmUAA2W; (total 64 bytes);
 17: SQL NULL;
 18: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 19: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 20: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 21: len 30; hex 7170477a4f42485647485a6c684e64616e635a346f7a724944673067494c; asc qpGzOBHVGHZlhNdancZ4ozrIDg0gIL; (total 64 bytes);
 22: len 9; hex 8000000000000fa000; asc          ;;
 23: len 4; hex 80000006; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d7e; asc    ~;;
 27: len 4; hex 80000018; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b0a1f; asc      ;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 800126ef; asc   & ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d39a; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 28 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9d; asc   : ;;
 1: len 6; hex 000001893fc3; asc     ? ;;
 2: len 7; hex 02000001a92239; asc      "9;;
 3: len 28; hex 6f546e44693541614967385f73513272683466515545376a6f356a41; asc oTnDi5AaIg8_sQ2rh4fQUE7jo5jA;;
 4: len 9; hex e5bca0e6a091e5b3b0; asc          ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 426a5a6d64375251366d7930534a2f516c426f346a673d3d; asc BjZmd7RQ6my0SJ/QlBo4jg==;;
 14: len 24; hex 3348745a4a4e626134615a586c7754515377565149773d3d; asc 3HtZJNba4aZXlwTQSwVQIw==;;
 15: len 24; hex 3272396444734632743943693669792f3864334264673d3d; asc 2r9dDsF2t9Ci6iy/8d3Bdg==;;
 16: len 30; hex 324d56347847537272726571756439454d7032334761704a4c722f4a6d55; asc 2MV4xGSrrrequd9EMp23GapJLr/JmU; (total 44 bytes);
 17: SQL NULL;
 18: len 24; hex 7945554a576434495a436f434f784876627a327358413d3d; asc yEUJWd4IZCoCOxHvbz2sXA==;;
 19: len 24; hex 4f694f3441304f2f7352616e4e4638506476336254513d3d; asc OiO4A0O/sRanNF8Pdv3bTQ==;;
 20: len 24; hex 32484f534c4872783547765650554f6f686a4b3064513d3d; asc 2HOSLHrx5GvVPUOohjK0dQ==;;
 21: len 30; hex 5079526d4e444b63504c2f3354504d717455574f2f3837706b77564f386e; asc PyRmNDKcPL/3TPMqtUWO/87pkwVO8n; (total 44 bytes);
 22: len 9; hex 80000000000032c800; asc       2  ;;
 23: len 4; hex 8000000a; asc     ;;
 24: len 24; hex 426a5a6d64375251366d7930534a2f516c426f346a673d3d; asc BjZmd7RQ6my0SJ/QlBo4jg==;;
 25: len 24; hex 3348745a4a4e626134615a586c7754515377565149773d3d; asc 3HtZJNba4aZXlwTQSwVQIw==;;
 26: len 4; hex 8000001f; asc     ;;
 27: len 4; hex 8000000a; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b136c; asc     l;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 80012784; asc   ' ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3731313836; asc 71186;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d399; asc     ;;
 44: SQL NULL;
 45: SQL NULL;

Record lock, heap no 29 PHYSICAL RECORD: n_fields 46; compact format; info bits 0
 0: len 4; hex 80003a9f; asc   : ;;
 1: len 6; hex 00000189457a; asc     Ez;;
 2: len 7; hex 020000019a1a8a; asc        ;;
 3: len 28; hex 6f546e446935475274695756694a4138635a6c486645304c61316634; asc oTnDi5GRtiWViJA8cZlHfE0La1f4;;
 4: len 9; hex e5b494e99baae88eb9; asc          ;;
 5: SQL NULL;
 6: SQL NULL;
 7: SQL NULL;
 8: SQL NULL;
 9: SQL NULL;
 10: SQL NULL;
 11: SQL NULL;
 12: SQL NULL;
 13: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 14: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 15: len 24; hex 5737395a2f4d613151666d54554d686f7378387a6a413d3d; asc W79Z/Ma1QfmTUMhosx8zjA==;;
 16: len 24; hex 4c6557783259574e4745546f532b6b66764846507a413d3d; asc LeWx2YWNGEToS+kfvHFPzA==;;
 17: SQL NULL;
 18: len 24; hex 6c75625152504c6c54586939304b582f384d4f4261513d3d; asc lubQRPLlTXi90KX/8MOBaQ==;;
 19: len 24; hex 76493841796967546a436249577058513467346a56773d3d; asc vI8AyigTjCbIWpXQ4g4jVw==;;
 20: len 24; hex 454d324d44336a37463438314d476948386e31485a673d3d; asc EM2MD3j7F481MGiH8n1HZg==;;
 21: len 30; hex 6253626434683462444a58474b6436506577557838495861655432326451; asc bSbd4h4bDJXGKd6PewUx8IXaeT22dQ; (total 44 bytes);
 22: len 9; hex 800000000000138800; asc          ;;
 23: len 4; hex 8000000a; asc     ;;
 24: len 24; hex 503069327a54424f774a3677574c6d786b4e4d7553673d3d; asc P0i2zTBOwJ6wWLmxkNMuSg==;;
 25: len 24; hex 4c6e792b6567377735654434724a51434f37473447773d3d; asc Lny+eg7w5eD4rJQCO7G4Gw==;;
 26: len 4; hex 80000d89; asc     ;;
 27: len 4; hex 80000001; asc     ;;
 28: SQL NULL;
 29: len 5; hex 99ae1b155a; asc     Z;;
 30: len 4; hex 80000002; asc     ;;
 31: SQL NULL;
 32: SQL NULL;
 33: SQL NULL;
 34: SQL NULL;
 35: SQL NULL;
 36: SQL NULL;
 37: SQL NULL;
 38: SQL NULL;
 39: len 4; hex 8001278b; asc   ' ;;
 40: len 4; hex 80000000; asc     ;;
 41: len 5; hex 3730373936; asc 70796;;
 42: len 4; hex 80000012; asc     ;;
 43: len 4; hex 8000d39c; asc     ;;
 44: SQL NULL;
 45: SQL NULL;


*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 1826 page no 266 n bits 80 index PRIMARY of table `ecnt`.`sys_user` trx id 25774543 lock_mode X waiting
Record lock, heap no 13 PHYSICAL RECORD: n_fields 29; compact format; info bits 64
 0: len 4; hex 8000d3a3; asc     ;;
 1: len 6; hex 0000018949d0; asc     I ;;
 2: len 7; hex 81000001bf0110; asc        ;;
 3: len 11; hex 3139393137373239393039; asc 19917729909;;
 4: SQL NULL;
 5: len 9; hex e69d8ee5ae87e888aa; asc          ;;
 6: SQL NULL;
 7: len 4; hex 80000000; asc     ;;
 8: len 4; hex 80000000; asc     ;;
 9: len 4; hex 80000000; asc     ;;
 10: SQL NULL;
 11: len 4; hex 80000003; asc     ;;
 12: len 11; hex 3139393137373239393039; asc 19917729909;;
 13: SQL NULL;
 14: len 17; hex 313938363430313731314071712e636f6d; asc 1986401711@qq.com;;
 15: SQL NULL;
 16: len 4; hex 8000001f; asc     ;;
 17: len 30; hex 457a454d334e73685337316c6966686c4f4c68794a6b6b42354848786843; asc EzEM3NshS71lifhlOLhyJkkB5HHxhC; (total 44 bytes);
 18: len 30; hex 36446d6e36764b6354346a7866534571305a58685a765843795773613136; asc 6Dmn6vKcT4jxfSEq0ZXhZvXCyWsa16; (total 44 bytes);
 19: len 30; hex 336154693335384c727a477433752b65746f5158622f4853674434384956; asc 3aTi358LrzGt3u+etoQXb/HSgD48IV; (total 44 bytes);
 20: len 3; hex 8f9c48; asc   H;;
 21: len 5; hex 99ae1b1821; asc     !;;
 22: len 4; hex 7fffffff; asc     ;;
 23: len 4; hex 80000002; asc     ;;
 24: SQL NULL;
 25: SQL NULL;
 26: SQL NULL;
 27: SQL NULL;
 28: len 5; hex 99be4c0000; asc   L  ;;

*** WE ROLL BACK TRANSACTION (1)
------------
TRANSACTIONS
------------
Trx id counter 26322245
Purge done for trx's n:o < 26314354 undo n:o < 0 state: running but idle
History list length 345
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 281475273551968, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273559728, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273558952, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273554296, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273560504, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273558176, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273533344, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273557400, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273556624, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273555848, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273549640, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273548864, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273547312, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273546536, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273543432, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273542656, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273531016, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273550416, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273548088, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273540328, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273534120, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273530240, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273578352, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273572920, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273570592, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273569040, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273568264, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273567488, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273566712, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273555072, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273553520, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273552744, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273538000, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273536448, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273535672, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273534896, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273562832, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273541880, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273532568, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273531792, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273551192, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273538776, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273544984, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273537224, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273544208, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273529464, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
---TRANSACTION 281475273528688, not started
0 lock struct(s), heap size 1128, 0 row lock(s)
--------
FILE I/O
--------
I/O thread 0 state: wait Windows aio (insert buffer thread)
I/O thread 1 state: wait Windows aio (log thread)
I/O thread 2 state: wait Windows aio (read thread)
I/O thread 3 state: wait Windows aio (read thread)
I/O thread 4 state: wait Windows aio (read thread)
I/O thread 5 state: wait Windows aio (read thread)
I/O thread 6 state: wait Windows aio (write thread)
I/O thread 7 state: wait Windows aio (write thread)
I/O thread 8 state: wait Windows aio (write thread)
I/O thread 9 state: wait Windows aio (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:
Pending flushes (fsync) log: 0; buffer pool: 39
26081611 OS file reads, 39353876 OS file writes, 8758902 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 16.91 writes/s, 10.41 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 4272, seg size 4274, 34859 merges
merged operations:
 insert 73372, delete mark 2669, delete 297
discarded operations:
 insert 0, delete mark 0, delete 0
Hash table size 1593833, node heap has 1859 buffer(s)
Hash table size 1593833, node heap has 277 buffer(s)
Hash table size 1593833, node heap has 2170 buffer(s)
Hash table size 1593833, node heap has 194 buffer(s)
Hash table size 1593833, node heap has 737 buffer(s)
Hash table size 1593833, node heap has 2374 buffer(s)
Hash table size 1593833, node heap has 2012 buffer(s)
Hash table size 1593833, node heap has 114 buffer(s)
18950.40 hash searches/s, 316.27 non-hash searches/s
---
LOG
---
Log sequence number          135712122398
Log buffer assigned up to    135712122398
Log buffer completed up to   135712122398
Log written up to            135712122398
Log flushed up to            135712122398
Added dirty pages up to      135712122398
Pages flushed up to          135712122398
Last checkpoint at           135712120033
7416666 log i/o's done, 10.20 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 0
Dictionary memory allocated 11178927
Buffer pool size   393168
Free buffers       8192
Database pages     375239
Old database pages 138353
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 220520597, not young 1424781565
33.56 youngs/s, 0.00 non-youngs/s
Pages read 26046509, created 14045967, written 28300862
0.00 reads/s, 0.19 creates/s, 4.40 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 375239, unzip_LRU len: 0
I/O sum[2792]:cur[0], unzip sum[0]:cur[0]
----------------------
INDIVIDUAL BUFFER POOL INFO
----------------------
---BUFFER POOL 0
Buffer pool size   49146
Free buffers       1024
Database pages     46907
Old database pages 17295
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 28291496, not young 177002900
4.68 youngs/s, 0.00 non-youngs/s
Pages read 3485194, created 1765077, written 3789901
0.00 reads/s, 0.00 creates/s, 0.57 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46907, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 1
Buffer pool size   49146
Free buffers       1024
Database pages     46888
Old database pages 17288
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27603495, not young 175541738
6.02 youngs/s, 0.00 non-youngs/s
Pages read 3490731, created 1756990, written 3588507
0.00 reads/s, 0.00 creates/s, 0.10 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 30 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46888, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 2
Buffer pool size   49146
Free buffers       1024
Database pages     46904
Old database pages 17294
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 26787335, not young 204293923
6.12 youngs/s, 0.00 non-youngs/s
Pages read 3160004, created 1742406, written 3836157
0.00 reads/s, 0.00 creates/s, 1.05 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46904, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 3
Buffer pool size   49146
Free buffers       1024
Database pages     46891
Old database pages 17289
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27695443, not young 150860970
6.12 youngs/s, 0.00 non-youngs/s
Pages read 3349522, created 1762771, written 3469901
0.00 reads/s, 0.00 creates/s, 0.48 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46891, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 4
Buffer pool size   49146
Free buffers       1024
Database pages     46912
Old database pages 17297
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27529428, not young 150753017
6.12 youngs/s, 0.00 non-youngs/s
Pages read 3007676, created 1746264, written 3436492
0.00 reads/s, 0.19 creates/s, 0.76 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 1 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46912, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 5
Buffer pool size   49146
Free buffers       1024
Database pages     46909
Old database pages 17296
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27704666, not young 179991923
4.30 youngs/s, 0.00 non-youngs/s
Pages read 3166416, created 1757792, written 3339485
0.00 reads/s, 0.00 creates/s, 0.48 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46909, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 6
Buffer pool size   49146
Free buffers       1024
Database pages     46911
Old database pages 17296
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27631891, not young 181439278
0.10 youngs/s, 0.00 non-youngs/s
Pages read 3218469, created 1759176, written 3500602
0.00 reads/s, 0.00 creates/s, 0.48 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46911, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
---BUFFER POOL 7
Buffer pool size   49146
Free buffers       1024
Database pages     46917
Old database pages 17298
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 27276843, not young 204897816
0.10 youngs/s, 0.00 non-youngs/s
Pages read 3168497, created 1755491, written 3339817
0.00 reads/s, 0.00 creates/s, 0.48 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 46917, unzip_LRU len: 0
I/O sum[349]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
3 read views open inside InnoDB
Process ID=4032, Main thread ID=4000 , state=sleeping
Number of rows inserted 3920245206, updated 445964, deleted 33334, read 593600300141
3.70 inserts/s, 3.10 updates/s, 0.00 deletes/s, 66261.07 reads/s
Number of system rows inserted 23699, updated 71451, deleted 22140, read 342914748
0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================
```

## SHOW ENGINE INNODB MUTEX

```log
+--------+------------------------------+---------------+
| Type   | Name                         | Status        |
+--------+------------------------------+---------------+
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=8       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=6       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=4       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=4       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=7       |
| InnoDB | rwlock: fil0fil.cc:3388      | waits=1       |
| InnoDB | rwlock: fil0fil.cc:3388      | waits=3       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=4       |
| InnoDB | rwlock: fil0fil.cc:3388      | waits=13      |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: fil0fil.cc:3388      | waits=19      |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=12      |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=3       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=2       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=3       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=7       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=3       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=1       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=3       |
| InnoDB | rwlock: dict0dict.cc:2537    | waits=4       |
| InnoDB | rwlock: fil0fil.cc:3388      | waits=119     |
| InnoDB | rwlock: dict0dict.cc:1024    | waits=30      |
| InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1       |
| InnoDB | rwlock: sync0sharded_rw.h:81 | waits=1       |
| InnoDB | rwlock: btr0sea.cc:202       | waits=42458   |
| InnoDB | rwlock: btr0sea.cc:202       | waits=168666  |
| InnoDB | rwlock: btr0sea.cc:202       | waits=220586  |
| InnoDB | rwlock: btr0sea.cc:202       | waits=87722   |
| InnoDB | rwlock: btr0sea.cc:202       | waits=822     |
| InnoDB | rwlock: btr0sea.cc:202       | waits=121916  |
| InnoDB | rwlock: btr0sea.cc:202       | waits=189444  |
| InnoDB | rwlock: btr0sea.cc:202       | waits=86834   |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=17      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=12      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=15      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=16      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=68      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=16      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=14      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=15      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=12      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=34      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=17      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=36      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=14      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=1       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=17      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=13      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=5       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=16      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=16      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=30      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=3       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=9       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=8       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=4       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=18      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=2       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=7       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=14      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=6       |
| InnoDB | rwlock: hash0hash.cc:163     | waits=10      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=21      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | rwlock: hash0hash.cc:163     | waits=11      |
| InnoDB | sum rwlock: buf0buf.cc:786   | waits=5775165 |
+--------+------------------------------+---------------+
171 rows in set (0.20 sec)
```

## performance_schema.events_waits_summary_global_by_event_name

```log
+---------------------------------------------------------+------------+-------------------+
| EVENT_NAME                                              | COUNT_STAR | SUM_TIMER_WAIT_MS |
+---------------------------------------------------------+------------+-------------------+
| wait/synch/mutex/innodb/trx_mutex                       |    1453287 | 22.2535           |
| wait/synch/mutex/innodb/lock_sys_page_mutex             |    1049399 | 11.3280           |
| wait/synch/mutex/innodb/fil_system_mutex                |     116518 | 7.4280            |
| wait/synch/mutex/innodb/log_writer_mutex                |      58697 | 12.2760           |
| wait/synch/mutex/innodb/log_write_notifier_mutex        |      58371 | 10.8139           |
| wait/synch/mutex/innodb/log_flush_notifier_mutex        |      58266 | 9.8795            |
| wait/synch/mutex/innodb/log_flusher_mutex               |      57153 | 11.9687           |
| wait/synch/mutex/innodb/srv_sys_mutex                   |      11814 | 5.4922            |
| wait/synch/mutex/innodb/flush_list_mutex                |      11755 | 0.9913            |
| wait/synch/mutex/innodb/buf_pool_LRU_list_mutex         |       9394 | 0.9230            |
| wait/synch/mutex/innodb/buf_pool_flush_state_mutex      |       8860 | 0.5636            |
| wait/synch/mutex/innodb/dblwr_mutex                     |       4482 | 0.5694            |
| wait/synch/mutex/innodb/page_cleaner_mutex              |       4344 | 0.7656            |
| wait/synch/mutex/innodb/srv_threads_mutex               |       4214 | 0.5219            |
| wait/synch/mutex/innodb/trx_sys_mutex                   |       3742 | 2.4002            |
| wait/synch/mutex/innodb/undo_space_rseg_mutex           |       3698 | 0.3567            |
| wait/synch/mutex/innodb/trx_sys_serialisation_mutex     |       2172 | 0.4515            |
| wait/synch/mutex/innodb/trx_sys_shard_mutex             |       1358 | 0.1514            |
| wait/synch/mutex/innodb/lock_sys_table_mutex            |       1210 | 0.3244            |
| wait/synch/mutex/innodb/log_limits_mutex                |       1057 | 0.0498            |
| wait/synch/mutex/innodb/trx_undo_mutex                  |        920 | 0.0434            |
| wait/synch/mutex/innodb/sync_array_mutex                |        873 | 0.0913            |
| wait/synch/mutex/innodb/autoinc_persisted_mutex         |        850 | 0.1254            |
| wait/synch/mutex/innodb/dict_sys_mutex                  |        849 | 0.0452            |
| wait/synch/mutex/innodb/autoinc_mutex                   |        822 | 0.0743            |
| wait/synch/mutex/innodb/dict_table_mutex                |        779 | 0.0419            |
| wait/synch/mutex/innodb/temp_space_rseg_mutex           |        768 | 0.0528            |
| wait/synch/mutex/innodb/innobase_share_mutex            |        605 | 0.0171            |
| wait/synch/mutex/innodb/lock_wait_mutex                 |        594 | 0.0947            |
| wait/synch/mutex/innodb/buf_pool_free_list_mutex        |        520 | 0.0201            |
| wait/synch/mutex/innodb/parallel_read_mutex             |        338 | 0.3216            |
| wait/synch/mutex/innodb/log_checkpointer_mutex          |        216 | 0.0661            |
| wait/synch/mutex/innodb/row_drop_list_mutex             |        198 | 0.0583            |
| wait/synch/mutex/innodb/ibuf_mutex                      |        170 | 0.0395            |
| wait/synch/mutex/innodb/dict_persist_dirty_tables_mutex |         60 | 0.0185            |
| wait/synch/mutex/innodb/srv_innodb_monitor_mutex        |         29 | 0.0104            |
| wait/synch/mutex/innodb/recalc_pool_mutex               |         19 | 0.0081            |
| wait/synch/mutex/innodb/page_sys_arch_oper_mutex        |         17 | 0.0051            |
| wait/synch/mutex/innodb/dict_foreign_err_mutex          |         13 | 0.0024            |
| wait/synch/mutex/innodb/trx_pool_mutex                  |          8 | 0.0015            |
| wait/synch/mutex/innodb/trx_pool_manager_mutex          |          4 | 0.0015            |
| wait/synch/mutex/innodb/ibuf_bitmap_mutex               |          1 | 0.0003            |
+---------------------------------------------------------+------------+-------------------+
```
