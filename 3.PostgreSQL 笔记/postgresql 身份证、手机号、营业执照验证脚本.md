- [postgresql 身份证、手机号、营业执照验证脚本](https://hanbo.blog.csdn.net/article/details/114115204)

# 验证18位身份证号码

```plsql
CREATE OR REPLACE FUNCTION "public"."check_idcard"("a_sfz" varchar)
  RETURNS "pg_catalog"."bool" AS $BODY$DECLARE
v_sfz  varchar;
v_i  integer;
v_sum  integer;
v_array1 integer[];
v_array2 varchar[];
v_s varchar;
BEGIN
	v_sfz:=upper(trim(a_sfz));
	raise notice '检测身份证号%',v_sfz;
	if length(v_sfz)=18 and  v_sfz ~ '^[123456789]\d{5}(19|20)\d{2}(0\d|10|11|12)(0\d|1\d|2\d|30|31)\d{3}[\dX]' then
		v_array1:=array[7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2];
		v_array2:=array['1','0','X','9','8','7','6','5','4','3','2'];
		v_i:=1;
		v_sum:=0;
		loop
			v_s:=substr(v_sfz,v_i,1);
			v_sum:=v_sum + cast(v_s as integer)*v_array1[v_i];
			v_i:=v_i + 1;
			if v_i>17 then
				exit;
			end if;
		end loop;
		v_sum:=mod(v_sum,11) + 1;
		v_s:=v_array2[v_sum];
		if v_s=substr(v_sfz,18,1) then
			return true;
		else
			return false;
		end if;
	else
			return false;
	end if;
END
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
```

# 验证手机号

```plsql
CREATE OR REPLACE FUNCTION "public"."check_phone"("name_per" text)
  RETURNS "pg_catalog"."bool" AS $BODY$
    ---------------------------------------------
-- 1.长度，2-11
-- 2.无特殊符号及数字、字母，·除外，^[\u4e00-\u9fa5]{0,}$
---------------------------------------------
 
declare
begin
 
    if length(name_per) !=11 then
        return false;
    else
        if replace(name_per,'·','') ~* '^1[3|4|5|6|7|8|9][0-9]\d{8}$' then
            return true;
        else
            return false;
        end if;
    end if;
end;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
```

# 验证营业执照

```plsql
CREATE OR REPLACE FUNCTION "public"."check_phone"("name_per" text)
  RETURNS "pg_catalog"."bool" AS $BODY$
    ---------------------------------------------
-- 1.长度，2-11
-- 2.无特殊符号及数字、字母，·除外，^[\u4e00-\u9fa5]{0,}$
---------------------------------------------
 
declare
begin
 
    if length(name_per) =0 then
        return false;
    else
        if replace(name_per,'·','') ~* '^[0-9A-HJ-NPQRTUWXY]{2}\d{6}[0-9A-HJ-NPQRTUWXY]{10}' then
            return true;
        else
            return false;
        end if;
    end if;
end;
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100
```