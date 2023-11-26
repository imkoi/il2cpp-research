# This document is describing reflection of managed objects in IL2CPP runtime
At introduction i would say that c++ code became more "managed",
because in c# we have reflection, that not exist in c++, so we need to have our VM to store reflection data and metadata
to access reflection and create objects. Reflection is allocated only when we will request it for type
and will be preserved in memory until application shutdown

## GetType

c#
```c#
private Type GetTypeExample()
{
    return GetType();
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR Type_t* TypeExample_GetTypeExample_m4BC95636C1E52F5DF10B7F37BA7A302D8ACCF0CD (TypeExample_tBE5BDE1E7519EB0E776ED504A80032ACF3F81615* __this, const RuntimeMethod* method) 
{
	{
		Type_t* L_0;
		//Foreach type we have custom GetType mehtod that cast our Type to Il2CppObject
		L_0 = Object_GetType_mE10A8FC1E57F3DF29972CCBC026C2DC3942263B3(__this, NULL);
		return L_0;
	}
}

//Whats called inside!
//WARNING! I skiped code inside Object_GetType_mE10A8FC1E57F3DF29972CCBC026C2DC3942263B3 because its just get Il2CppType from object
Il2CppReflectionType* Reflection::GetTypeObject(const Il2CppType *type) 
{
    Il2CppReflectionType* object = NULL;

    // Trying to get Type from cache, if we previously access it
    if (s_TypeMap->TryGetValue(type, &object))
        return object;

    // No type - so lets allocate new one
    Il2CppReflectionType* typeObject = (Il2CppReflectionType*)Object::New(il2cpp_defaults.runtimetype_class);

    typeObject->type = type;

    //Put type to cache map that will be stored in memory until we will shutdown our application
    return s_TypeMap->GetOrAdd(type, typeObject);
}
```

## typeof

c#
```c#
private Type TypeOfExample()
{
    return typeof(TypeExample);
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR Type_t* TypeExample_TypeOfExample_mB3CAD2DF34865E7D12ACEAE64BFD9B5EC339C70E (TypeExample_tBE5BDE1E7519EB0E776ED504A80032ACF3F81615* __this, const RuntimeMethod* method) 
{
	static bool s_Il2CppMethodInitialized;
	if (!s_Il2CppMethodInitialized)
	{
		//1st overhead - we need a metadata to get the type from, it will be applied only on first method call
		il2cpp_codegen_initialize_runtime_metadata((uintptr_t*)&TypeExample_tBE5BDE1E7519EB0E776ED504A80032ACF3F81615_0_0_0_var);
		s_Il2CppMethodInitialized = true;
	}
	{
		//Cast to runtimetypehandler
		RuntimeTypeHandle_t332A452B8B6179E4469B69525D0FE82A88030F7B L_0 = { reinterpret_cast<intptr_t> (TypeExample_tBE5BDE1E7519EB0E776ED504A80032ACF3F81615_0_0_0_var) };
		//2nd overhead - we initialize class object for systemtype_class
		il2cpp_codegen_runtime_class_init_inline(il2cpp_defaults.systemtype_class);
		Type_t* L_1;
		// Get type from handle
		L_1 = Type_GetTypeFromHandle_m6062B81682F79A4D6DF2640692EE6D9987858C57(L_0, NULL);
		return L_1;
	}
}

IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR Type_t* Type_GetTypeFromHandle_m6062B81682F79A4D6DF2640692EE6D9987858C57 (RuntimeTypeHandle_t332A452B8B6179E4469B69525D0FE82A88030F7B ___0_handle, const RuntimeMethod* method) 
{
	{
		intptr_t L_0;
		// put handle value to L_0
		L_0 = RuntimeTypeHandle_get_Value_mDC366CF36C3E21505134EAEE72BD7629107D762A_inline((&___0_handle), NULL);
		bool L_1;

		//L_1 = handle value is null
		L_1 = IntPtr_op_Equality_m7D9CDCDE9DC2A0C2C614633F4921E90187FAB271_inline(L_0, 0, NULL);

		//goto IL_0015 if handle vlaue is not null
		if (!L_1)
		{
			goto IL_0015;
		}
	}
	{
		//else return null type, in this case application will crash, because we dont check Type_t object for null
		return (Type_t*)NULL;
	}

IL_0015:
	{
		intptr_t L_2;
		L_2 = RuntimeTypeHandle_get_Value_mDC366CF36C3E21505134EAEE72BD7629107D762A_inline((&___0_handle), NULL);
		//No overhead, since we inited it previously
		il2cpp_codegen_runtime_class_init_inline(il2cpp_defaults.systemtype_class);
		Type_t* L_3;

		//Get type from runtimehandler
		L_3 = Type_internal_from_handle_m0747FFFEFFCA22D764E069CA2EC64E666294D7E5(L_2, NULL);
		return L_3;
	}
}

Il2CppReflectionType* Type::GetTypeFromHandle(intptr_t handle)
{
    const Il2CppType* type = (const Il2CppType*)handle;
    //get class object that store metadata from type
    Il2CppClass *klass = vm::Class::FromIl2CppType(type);

    //Get type object
    return il2cpp::vm::Reflection::GetTypeObject(&klass->byval_arg);
}

Il2CppReflectionType* Reflection::GetTypeObject(const Il2CppType *type) 
{
    Il2CppReflectionType* object = NULL;

    // Trying to get Type from cache, if we previously access it
    if (s_TypeMap->TryGetValue(type, &object))
        return object;

    // No type - so lets allocate new one
    Il2CppReflectionType* typeObject = (Il2CppReflectionType*)Object::New(il2cpp_defaults.runtimetype_class);

    typeObject->type = type;

    //Put type to cache map that will be stored in memory until we will shutdown our application
    return s_TypeMap->GetOrAdd(type, typeObject);
}
```

So! If you have an ability to use GetType instead of typeof - do it, because you will 
