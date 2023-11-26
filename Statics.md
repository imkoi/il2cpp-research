# This document is describing statics in IL2CPP runtime
In this document i would cover how IL2CPP using static references that used in game

## Method without static variable usage

c#
```c#
public static bool StringExtension(this string myString)
{
    return string.IsNullOrEmpty(myString);
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR bool ExtensionExample_StringExtension_m8C1F043FDC8431B93D233400B6FB20D05CE0F477 (String_t* ___0_myString, const RuntimeMethod* method) 
{
	{
		String_t* L_0 = ___0_myString;
		bool L_1;
		L_1 = String_IsNullOrEmpty_mEA9E3FB005AC28FE02E69FCF95A7B8456192B478(L_0, NULL);
		return L_1;
	}
}
```

## Method with static variable usage

c#
```c#
public static bool StringExtensionsWitDep(this string myString)
{
    _myArray[0] = myString; // _myArray is stored as static variable in class
    return false;
}
```

c++
```c++
static bool s_Il2CppMethodInitialized;
	if (!s_Il2CppMethodInitialized)
	{
	    //fill metadata to ExtensionExample_tD9091413DE17B1108BF02674611D28465BA9A7B0_il2cpp_TypeInfo_var
		il2cpp_codegen_initialize_runtime_metadata((uintptr_t*)&ExtensionExample_tD9091413DE17B1108BF02674611D28465BA9A7B0_il2cpp_TypeInfo_var);
		s_Il2CppMethodInitialized = true;
	}
	{
	    // initialize class object if not initlaized before
		il2cpp_codegen_runtime_class_init_inline(ExtensionExample_tD9091413DE17B1108BF02674611D28465BA9A7B0_il2cpp_TypeInfo_var);
		//get array reference from our class
		StringU5BU5D_t7674CD946EC0CE7B3AE0BE70E6EE85F2ECD9F248* L_0 = ((ExtensionExample_tD9091413DE17B1108BF02674611D28465BA9A7B0_StaticFields*)il2cpp_codegen_static_fields_for(ExtensionExample_tD9091413DE17B1108BF02674611D28465BA9A7B0_il2cpp_TypeInfo_var))->____myArray;
		String_t* L_1 = ___0_myString;
		NullCheck(L_0);
		(L_0)->SetAt(static_cast<il2cpp_array_size_t>(0), (String_t*)L_1);
		return (bool)0;
	}
}
```

So, if our static method is using any static variable from class:
* it will get metadata of this class
* It will try to initialize our class - allocate our static objects of class
* Make null check before usage