报错：
```
pre path:  /home/leejoy/workspace/timelyreapi/scripts
cur path:  /home/leejoy/workspace/timelyreapi/test/leopard_api_unit_te
st
>>>>>>>>> leopard api unit_tests
============================= test session starts ====================
==========
platform linux -- Python 3.9.18, pytest-7.2.2, pluggy-1.3.0 -- /home/l
eejoy/miniconda3/envs/leopard/bin/python
cachedir: .pytest_cache
metadata: {'Python': '3.9.18', 'Platform': 'Linux-6.7.6-arch1-2-x86_64
-with-glibc2.39', 'Packages': {'pytest': '7.2.2', 'pluggy': '1.3.0'}, 
'Plugins': {'env': '1.1.3', 'metadata': '3.0.0', 'pythonpath': '0.7.3'
, 'json-report': '1.5.0', 'xdist': '3.5.0'}, 'JAVA_HOME': '/home/leejo
y/.sdkman/candidates/java/current'}
rootdir: /home/leejoy/workspace/timelyreapi
plugins: env-1.1.3, metadata-3.0.0, pythonpath-0.7.3, j
0, xdist-3.5.0
16 workers [1 item]       
scheduling tests via LoadScheduling

series_groupby_api/test_agg.py::TestAggCases::test_agg_
unc_or_funcs0] 
[gw0] FAILED series_groupby_api/test_agg.py::TestAggCas
nc_or_funcs[func_or_funcs0] 

=================================== FAILURES ==========
=========================
_____________ TestAggCases.test_agg_func_or_funcs[func_
or_funcs0] ______________
[gw0] linux -- Python 3.9.18 /home/leejoy/miniconda3/en
vs/leopard/bin/python

self = <test.leopard_api_unit_test.series_groupby_api.t
est_agg.TestAggCases object at 0x7f2e638863d0>
func_or_funcs = ['count']

    def test_agg_func_or_funcs(self, func_or_funcs: list):
        expected_df = (
            self.pd_df.groupby("code")["volume"].agg(func_or_funcs).sort_index()
        )
        result_df = (
>           self.df.groupby("code")["volume"]
            .agg(func_or_funcs)
            .to_pandas()
            .sort_index()
        )

series_groupby_api/test_agg.py:44: 

_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

s = 'progress'

    @staticmethod
    def from_string(s):
        ss = s.split(",")
>       return int(ss[0]), int(ss[1]), int(ss[2]), int(ss[3])
E       ValueError: invalid literal for int() with base 10: 'progress'

../../src/transwarp_df/sql/connect/progress_bar/stage_progress.py:12: ValueError
'run_simple_command'(("insert into `api_test_db`.`src_table_d9b6a69adb8f11ee819da029425bd844` values ('0.CN','37.36147902649403','2022-01-01 0
9:30:00','993','-993.0','0.3736147902649403','True','-993','2022-01-01 09:30:00'),('0.CN','36.87829349127158','2022-01-01 09:30:03','289','-289.0','0.3687829349127158','False','-289','202
2-01-01 09:30:03'),('0.CN','-36.479516723947945','2022-01-01 09:30:06','-2','-2.0','0.36479516723947947','True','2','2022-01-01 09:30:06'),('0.CN','36.95854528305717','2022-01-01 09:30:09
','-401','401.0','0.3695854528305717','True','401','2022-01-01 09:30:09'),('0.CN','-36.45882269346517','2022-01-01 09:30:12','617','617.0','0.3645882269346517','True','-617','2022-01-01 0
9:30:12'),('1.CN','70.57475032283004','2022-01-01 09:30:00','-183','-183.0','0.7057475032283004','True','-183','2022-01-01 09:30:00'),('1.CN','-70.43870709226191','2022-01-01 09:30:03','2
88','288.0','-0.7043870709226191','False','288','2022-01-01 09:30:03'),('1.CN','-70.65902279825275','2022-01-01 09:30:06','-27','-27.0','-0.7065902279825275','True','27','2022-01-01 09:30
:06'),('1.CN','70.77255832495513','2022-01-01 09:30:09','-461','461.0','0.7077255832495513','True','-461','2022-01-01 09:30:09'),('1.CN','-70.96196614158436','2022-01-01 09:30:12','-12','
-12.0','-0.7096196614158435','False','12','2022-01-01 09:30:12'),('2.CN','-92.31374501752171','2022-01-01 09:30:00','574','-574.0','0.9231374501752171','True','-574','2022-01-01 09:30:00'
),('2.CN','-92.22663508863354','2022-01-01 09:30:03','-963','-963.0','-0.9222663508863355','False','-963','2022-01-01 09:30:03'),('2.CN','92.06514627743177','2022-01-01 09:30:06','948','9
48.0','-0.9206514627743176','False','948','2022-01-01 09:30:06'),('2.CN','-92.28812659994043','2022-01-01 09:30:09','-409','409.0','0.9228812659994042','True','409','2022-01-01 09:30:09')
,('2.CN','-92.06290652566952','2022-01-01 09:30:12','-345','-345.0','-0.9206290652566952','True','345','2022-01-01 09:30:12')",), {}) executed in 0.12 seconds
INFO     database:base_db_class.py:13 Method 'insert'(('src_table_d9b6a69adb8f11ee819da029425bd844', [['0.CN', 37.36147902649403, '2022-01-01 09:30:00', 993, -993.0, 0.3736147902649403, T
rue, -993, '2022-01-01 09:30:00'], ['0.CN', 36.87829349127158, '2022-01-01 09:30:03', 289, -289.0, 0.3687829349127158, False, -289, '2022-01-01 09:30:03'], ['0.CN', -36.479516723947945, '
2022-01-01 09:30:06', -2, -2.0, 0.36479516723947947, True, 2, '2022-01-01 09:30:06'], ['0.CN', 36.95854528305717, '2022-01-01 09:30:09', -401, 401.0, 0.3695854528305717, True, 401, '2022-
01-01 09:30:09'], ['0.CN', -36.45882269346517, '2022-01-01 09:30:12', 617, 617.0, 0.3645882269346517, True, -617, '2022-01-01 09:30:12'], ['1.CN', 70.57475032283004, '2022-01-01 09:30:00'
, -183, -183.0, 0.7057475032283004, True, -183, '2022-01-01 09:30:00'], ['1.CN', -70.43870709226191, '2022-01-01 09:30:03', 288, 288.0, -0.7043870709226191, False, 288, '2022-01-01 09:30:
03'], ['1.CN', -70.65902279825275, '2022-01-01 09:30:06', -27, -27.0, -0.7065902279825275, True, 27, '2022-01-01 09:30:06'], ['1.CN', 70.77255832495513, '2022-01-01 09:30:09', -461, 461.0
, 0.7077255832495513, True, -461, '2022-01-01 09:30:09'], ['1.CN', -70.96196614158436, '2022-01-01 09:30:12', -12, -12.0, -0.7096196614158435, False, 12, '2022-01-01 09:30:12'], ['2.CN', 
-92.31374501752171, '2022-01-01 09:30:00', 574, -574.0, 0.9231374501752171, True, -574, '2022-01-01 09:30:00'], ['2.CN', -92.22663508863354, '2022-01-01 09:30:03', -963, -963.0, -0.922266
3508863355, False, -963, '2022-01-01 09:30:03'], ['2.CN', 92.06514627743177, '2022-01-01 09:30:06', 948, 948.0, -0.9206514627743176, False, 948, '2022-01-01 09:30:06'], ['2.CN', -92.28812
659994043, '2022-01-01 09:30:09', -409, 409.0, 0.9228812659994042, True, 409, '2022-01-01 09:30:09'], ['2.CN', -92.06290652566952, '2022-01-01 09:30:12', -345, -345.0, -0.9206290652566952
, True, 345, '2022-01-01 09:30:12']], 10000), {'parallelism': 8, 'run_mode': 'crux'}) executed in 3.10 seconds


/home/leejoy/workspace/timelyreapi/src/transwarp_df/sql/connect/progress_bar/progress_tracer.py:56: UserWarning: tracer got exception, maybe the leopard version is not supported for tra
cer
    warnings.warn(f"tracer got exception, maybe the leopard version is not supported for tracer")

-- Docs: https://docs.pytest.org/en/stable/
- generated xml file: /home/leejoy/workspac
--------------------------------- JSON repo
report saved to: .report.json
=========================== short test summ
FAILED series_groupby_api/test_agg.py::Test
======================= 1 failed, 121 warni
[ leopard_api_unit_test ] summary: 
 Counter({'failed': 1, 'total': 1, 'collect


```