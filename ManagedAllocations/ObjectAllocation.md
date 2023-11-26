# This document is describing allocation of managed objects in IL2CPP runtime

### Object allocation

c#
```c#
private object AllocateObject()
{
    return new object();
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR RuntimeObject* ObjectExample_AllocateObject_m39A2C1512DA15A8A1F912B1A885210F40FE0C732 (ObjectExample_t71B844183E70F2081B42885E8E348D205AC2CB12* __this, const RuntimeMethod* method) 
{
	static bool s_Il2CppMethodInitialized;
	if (!s_Il2CppMethodInitialized)
	{
		//Here will be allocated all the metadata that is needed by this mehtod
		//Here we need RuntimeObject_il2cpp_TypeInfo_var, because we would tell il2cpp which object we would allocate
		//RuntimeObject_il2cpp_TypeInfo_var variable that describe class of "System.Object" and used in il2cpp as identifyer

		//il2cpp_codegen_initialize_runtime_metadata - create runtime metadata for "System.Object"
		// In metadata we will have class that has all Reflection data (Type, MethodsInfos, PropertyInfos)
		//Reflection data allocated one time when you will try to access it, after that it will be stored in Il2CppClass object
		
		il2cpp_codegen_initialize_runtime_metadata((uintptr_t*)&RuntimeObject_il2cpp_TypeInfo_var);

		// Dont allocate this on second call of method
		s_Il2CppMethodInitialized = true;
	}
	{
		// Create object for RuntimeObject_il2cpp_TypeInfo_var ("System.Object") 
		RuntimeObject* L_0 = (RuntimeObject*)il2cpp_codegen_object_new(RuntimeObject_il2cpp_TypeInfo_var);

		//Call constructor of class
		Object__ctor_mE837C6B9FA8C6D5D109F4B2EC885D79919AC0EA2(L_0, NULL);
		return L_0;
	}
}
```

### Object allocation and accessing:

c#
```c#
private object AllocateAndAccessObject()
{
    var obj = new object();

    obj.GetHashCode();
    obj.GetType();

    return obj;
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR RuntimeObject* ObjectExample_AllocateAndAccessObject_m2EAA3AB91F5B1272AA70C0E1B4E5A470D249B221 (ObjectExample_t71B844183E70F2081B42885E8E348D205AC2CB12* __this, const RuntimeMethod* method) 
{
	static bool s_Il2CppMethodInitialized;
	if (!s_Il2CppMethodInitialized)
	{
		//As we allocating new object we need to have its metadata first
		il2cpp_codegen_initialize_runtime_metadata((uintptr_t*)&RuntimeObject_il2cpp_TypeInfo_var);
		s_Il2CppMethodInitialized = true;
	}
	{
		RuntimeObject* L_0 = (RuntimeObject*)il2cpp_codegen_object_new(RuntimeObject_il2cpp_TypeInfo_var);
		Object__ctor_mE837C6B9FA8C6D5D109F4B2EC885D79919AC0EA2(L_0, NULL);
		RuntimeObject* L_1 = L_0;
		//Everytime we access any managed object - we will check it for null, to preverse crash of application
		NullCheck(L_1);
		int32_t L_2;
		L_2 = VirtualFuncInvoker0< int32_t >::Invoke(2, L_1);
		RuntimeObject* L_3 = L_1;
		//Everytime we access any managed object - we will check it for null, to preverse crash of application
		NullCheck(L_3);
		Type_t* L_4;
		L_4 = Object_GetType_mE10A8FC1E57F3DF29972CCBC026C2DC3942263B3(L_3, NULL);
		return L_3;
	}
}
```

