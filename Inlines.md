# This document is describing allocation of managed objects in IL2CPP runtime

### NOT inlined method

c#
```c#
private int NoInlineExampleMethod()
{
    var sum = 0;
            
    for (var i = 0; i < 1000000; i++)
    {
        sum += i;
    }

    return sum;
}
```

c++
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR int32_t InlineExample_NoInlineExampleMethod_mB70CFD0329174B1EE4B92CE4C173B1F10C0543B3 (InlineExample_t044D1BE0A3676DBBF666C38EF554BA742A740A99* __this, const RuntimeMethod* method) 
{
	//Here we dont use any class metadata, since we dont use it
	
	int32_t V_0 = 0;
	int32_t V_1 = 0;
	{
		V_0 = 0;
		V_1 = 0;
		goto IL_000e; // il2cpp uses goto operator to make any kind of loops
	}

IL_0006: // sum of i and sum variables
	{
		int32_t L_0 = V_0;
		int32_t L_1 = V_1;
		V_0 = ((int32_t)il2cpp_codegen_add(L_0, L_1)); 
		int32_t L_2 = V_1;
		V_1 = ((int32_t)il2cpp_codegen_add(L_2, 1));
	}

IL_000e:
	{
		int32_t L_3 = V_1;
		if ((((int32_t)L_3) < ((int32_t)((int32_t)1000000)))) // check condition
		{
			goto IL_0006; // goto IL_0006 block
		}
	}
	{
		int32_t L_4 = V_0;
		return L_4;
	}
}
```

### Inlined method

c#
```c#
[MethodImpl(MethodImplOptions.AggressiveInlining)]
private int InlineExampleMethod()
{
    var sum = 0;
            
    for (var i = 0; i < 1000000; i++)
    {
        sum += i;
    }

    return sum;
}
```

And here its tricky, because il2cpp generate two implementation for this method.
One is inlined and second one is not

Not Inlined version:
```c++
//WARNING! NOT INLINED!
//In this method we have no Inline macros, so c++ compiler will not event try to inline it
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR int32_t InlineExample_InlineExampleMethod_mB481FAED9063537EB69A3293DBBAE2B4F6C34060 (InlineExample_t044D1BE0A3676DBBF666C38EF554BA742A740A99* __this, const RuntimeMethod* method) 
{
	int32_t V_0 = 0;
	int32_t V_1 = 0;
	{
		V_0 = 0;
		V_1 = 0;
		goto IL_000e;
	}

IL_0006:
	{
		int32_t L_0 = V_0;
		int32_t L_1 = V_1;
		V_0 = ((int32_t)il2cpp_codegen_add(L_0, L_1));
		int32_t L_2 = V_1;
		V_1 = ((int32_t)il2cpp_codegen_add(L_2, 1));
	}

IL_000e:
	{
		int32_t L_3 = V_1;
		if ((((int32_t)L_3) < ((int32_t)((int32_t)1000000))))
		{
			goto IL_0006;
		}
	}
	{
		int32_t L_4 = V_0;
		return L_4;
	}
}
```

Not inlined:
```c++
//In this method we have IL2CPP_MANAGED_FORCE_INLINE, so c++ compiler will try to inline it!
IL2CPP_MANAGED_FORCE_INLINE IL2CPP_METHOD_ATTR int32_t InlineExample_InlineExampleMethod_mB481FAED9063537EB69A3293DBBAE2B4F6C34060_inline (InlineExample_t044D1BE0A3676DBBF666C38EF554BA742A740A99* __this, const RuntimeMethod* method) 
{
	int32_t V_0 = 0;
	int32_t V_1 = 0;
	{
		V_0 = 0;
		V_1 = 0;
		goto IL_000e;
	}

IL_0006:
	{
		int32_t L_0 = V_0;
		int32_t L_1 = V_1;
		V_0 = ((int32_t)il2cpp_codegen_add(L_0, L_1));
		int32_t L_2 = V_1;
		V_1 = ((int32_t)il2cpp_codegen_add(L_2, 1));
	}

IL_000e:
	{
		int32_t L_3 = V_1;
		if ((((int32_t)L_3) < ((int32_t)((int32_t)1000000))))
		{
			goto IL_0006;
		}
	}
	{
		int32_t L_4 = V_0;
		return L_4;
	}
}
```

How we call it
```c++
IL2CPP_EXTERN_C IL2CPP_METHOD_ATTR void InlineExample_Awake_m65BC047E9602B3D30C59DD8AA63F9993792C09B7 (InlineExample_t044D1BE0A3676DBBF666C38EF554BA742A740A99* __this, const RuntimeMethod* method) 
{
	{
		int32_t L_0;
		L_0 = InlineExample_NoInlineExampleMethod_mB70CFD0329174B1EE4B92CE4C173B1F10C0543B3(__this, NULL);
		int32_t L_1;
		// Foof! And here we use inlined version of this method, so stay calm!
		L_1 = InlineExample_InlineExampleMethod_mB481FAED9063537EB69A3293DBBAE2B4F6C34060_inline(__this, NULL);
		return;
	}
}
```

I dont know why il2cpp generates two version of method,
because as I understand method that cannot be inlined because of recursions and scary loops - should will be not inlined in asm
