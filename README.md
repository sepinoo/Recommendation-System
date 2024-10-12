# Recommendation-System
Recommendation System
جهت آموزش مجدد (retrain) مدل سيستم پيشنهاد دهنده كه معمولا بهتر است هر 3 ماه يكبار اتفاق بيفتد مراحل زير را انجام دهيد:
•	وارد لينك http://172.31.13.60:8000/ شده و نام كاربري: gc-ai و رمز عبور: GC@AI2022#  را وارد كنيد.
•	دو فايل code1000-Copy1.ipynb و code1000-Copy2.ipynb به ترتيب براي آموزش داده هاي مشتريان كلانشهرها و مشتريان غيركلان شهرها و در نهايت ادغام پيشنهادات به اين مشتريان هستند. ابتدا فايل code1000-Copy1.ipynb را باز كرده و به ترتيب cell هاي مربوطه را اجرا كنيد (Ctrl+ Enter)
•	در cell مربوط به محصولات فعال، فايل products20.csv را با محصولات فعال كه با كوئري زير اجرا مي شوند به روزرساني كرده و به جاي فايل قبلي جايگزين كنيد:
SELECT
        f.Product_Code as products
        --p.ProductTitle,
       -- COUNT(*) AS Count
FROM Sale.FactV4 (NOLOCK) f
JOIN Inventory.DimProduct p ON f.Product_Code = p.ProductBK
WHERE Order_Date BETWEEN 14011101 AND 14020631 
GROUP BY
        f.Product_Code,
        p.ProductTitle
 having f.Product_Code not in
 --59 ta mahsool dar baze 6 mahe gozashte jadid boodand
 (select top 59  ProductBK from Inventory.DimProduct
 order by RegisteredDate desc)
 and  count(*)>530


•	در cell مربوط به "خواندن لیست مشتریانی که تا کنون تراکنش داشته اند به همراه 8 فیچر آنها"، فايل CustomerWithTransactions.csv را با كوئري زير به روزرساني و جايگزين كنيد:
SELECT
  dc.CustomerBk,
		dc.AreaBk,
        dc.LastCustomerClassBk,       
        dc.CustomerTypeBk,
		dc.DistPathCode,
        dc.OfficeBk,
		dc.CityBk,
        ci.CityTitle,
        so.Actived
FROM  [DWAI].[Sale].[DimCustomer] dc 
JOIN [DWAI].[regional].[DimSaleOffice] so ON dc.OfficeBk = so.SaleOfficeBk
JOIN [DWAI].[regional].[DimCity] ci on ci.CityBK = dc.CityBk
where  dc.DeactiveDate IS NULL
            AND dc.CustomerBk IN (
                    SELECT CustomerCode
                    FROM [DWAI].[Sale].[FactV4]
                )

•	در cell مربوط به "خواندن لیست مشتریانی که تا کنون تراکنش نداشته اند به همراه 8 فیچر آنها"، فايل CustomerWithNoTransactions.csv را با كوئري زير به روزرساني و جايگزين كنيد:
SELECT
        dc.CustomerBk,
		dc.AreaBk,
        dc.LastCustomerClassBk,       
        dc.CustomerTypeBk,
		dc.DistPathCode,
        dc.OfficeBk,
		dc.CityBk,
        ci.CityTitle,
        so.Actived
              	  
FROM  [DWAI].[Sale].[DimCustomer] dc 
JOIN [DWAI].[regional].[DimSaleOffice] so ON dc.OfficeBk = so.SaleOfficeBk
JOIN [DWAI].[regional].[DimCity] ci on ci.CityBK = dc.CityBk
where  dc.DeactiveDate IS NULL
            AND dc.CustomerBk not IN (
                    SELECT CustomerCode
                    FROM [DWAI].[Sale].[FactV4]
                )
•	به ليست مشتريان در اين فايل ها با batch هاي 10000 تايي پيشنهادات ارائه مي شود و در part هاي مختلف ذخيره و در نهايت ادغام مي شوند.
•	فايل دوم نيز به همين ترتيب براي مشتريان غير كلانشهر اجرا مي شود.
•	فايل نهايي مشتريان و پيشنهادات آنها در آخرين cell فايل دوم بعنوان مثال با نام recommender_final_14020209.csv ذخيره مي شود. 
•	اين فايل با كوئري موجود در فولدر D:\Taghvaei\Recommendation System Codes\T SQL به نام UnpivotModelResult.sql بايد Unpivot شود و در نهايت در جدولWRS. [dbo].[Recommendation_Model_Output] جايگزين شود.
•	پيشنهادات جديد به همراه تاريخ train مدل بايد در جدول هيستوري با كوئري زير ذخيره گردد:
insert into [dbo].[Recommendation_Model_Output_History]([CustomerCode],[ProductCode],[Rank],[InsertDate])
(select [CustomerCode],[ProductCode],[Rank],[TrainDate] from [dbo].[Recommendation_Model_Output] )
•	تاريخ  NextTrainDate از جدول  RS. Recommendation_Model_Output_History  بايد با كوئري زير به روز رساني گردد:
update [dbo].[Recommendation_Model_Output_History] set NextTrainDate='تاريخ ميلادي ترين جديد'
where InsertDate='تاريخ ميلادي ترين قبلي'

•	توجه شود كه در جدول   براي  InsertDate= تاريخ ترين جديد، فيلد NextTrainDate بايد Null باشد.
•	تاريخ آموزش مدل در جدول WRS. Recommendation_Model_Output_History با فيلد InsertDate ذخيره مي شود.

