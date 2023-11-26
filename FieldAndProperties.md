# This document is describing field and variables in IL2CPP runtime

c#
```c#
public class VariablesExample
{
    public int Field;
    public int PropertyEmpty { get; set; }
    public int PropertyWithLambda => 4;
    public int PropertyWithExpressionFromField => Field;
    public int PropertyReferenceOtherProperties => PropertyWithExpressionFromField + PropertyEmpty;
    public int PropertyWithLoop
    {
        get
        {
            var sum = 10;
            
            for (var i = 0; i < _propertyWithLoopValue; i++)
            {
                sum++;
            }

            return sum;
        }
        set => _propertyWithLoopValue = value;
    }
    
    public int PropertyWithLoopInlined
    {
        get => GetPropertyWithLoop();
        set => _propertyWithLoopValue = value;
    }
    
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    private int GetPropertyWithLoop()
    {
        return 0;
    }

    private int _propertyWithLoopValue;

    public void PropertiesExampleMethod()
    {
        Field = 10;
        var fieldValue = Field;
        PropertyEmpty = 10;
        var propertyEmptyValue = PropertyEmpty;
        var propertyWithLambdaValue = PropertyWithLambda;
        var propertyWithLambdaToField = PropertyWithExpressionFromField;
        var propertyReferenceOtherProperties = PropertyReferenceOtherProperties;
        PropertyWithLoop = 10;
        var propertyWithLoop = PropertyWithLoop;
        PropertyWithLoopInlined = 10;
        var propertyWithLoopInlined = PropertyWithLoopInlined;
    }
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR void VariablesExample_PropertiesExampleMethod_m373F1EA0C13D144B31BF8310B527909716C544B8 (VariablesExample_t0D90716BA08200AA1714EBF39C4DC581C5451651* __this, const RuntimeMethod* method) 
{
	{
		//Fields has direct set/get
		__this->___Field = ((int32_t)10);
		int32_t L_0 = __this->___Field;
		
		//Same for default property with get;set;, because its inlined, and under the hood it set value in the type field
		VariablesExample_set_PropertyEmpty_m6F00D9C70576C4E62BDD68B7C5CC0B594B94E461_inline(__this, ((int32_t)10), NULL);
		int32_t L_1;
		//Still good, since property is inlined
		L_1 = VariablesExample_get_PropertyEmpty_mF1190684DB6FCEB96F614597E86B380057698C84_inline(__this, NULL);

		//Here we not use inlined method call, so we will have overhead to get value from method return
		int32_t L_2;
		L_2 = VariablesExample_get_PropertyWithLambda_m1B8991F0151251082AAB6F5BCC6CAEABDAC3CC94(__this, NULL);

		//If property expression return field - get call will be inlined
		int32_t L_3;
		L_3 = VariablesExample_get_PropertyWithExpressionFromField_m35A665A5BE2444EDBB7AFE54E755F90155C66103_inline(__this, NULL);
		
		//Not inlined becuase one of properties are not inlined
		int32_t L_4;
		L_4 = VariablesExample_get_PropertyReferenceOtherProperties_m4C95C4C7CC31363080A100B480C2E7DF1CD02EF5(__this, NULL);

		//Inlined because we set value to field inside
		VariablesExample_set_PropertyWithLoop_mDB406EAD8F356741A4E7DADA91F05B52649B2FF2_inline(__this, ((int32_t)10), NULL);

		//Not Inlined because its not return field value
		int32_t L_5;
		L_5 = VariablesExample_get_PropertyWithLoop_m0000C49DCD990B19E33268626B64E259D2C2BC46(__this, NULL);

		//Inlined because we set value to field inside
		VariablesExample_set_PropertyWithLoopInlined_mDAF8E7AC94CB9C547E89E67F24FAB16F9A9157AA_inline(__this, ((int32_t)10), NULL);

		//Even if getter of property is inlined we will use NOT inlined version of it here
		int32_t L_6;
		L_6 = VariablesExample_get_PropertyWithLoopInlined_m7606B1DD09D27288E1ABB3C673A028983CAB0496(__this, NULL);
		return;
	}
}
```

Depend on code we will have this behaviour of variables:
* Fields calls is always direct
* Property calls will be inlined if we return or set value into class field
* Property calls will be NOT inlined if we return value from expression body
* Default int Property { get; set; } will be inlined! So no performance difference with fields
