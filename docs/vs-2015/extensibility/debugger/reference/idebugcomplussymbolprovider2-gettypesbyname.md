---
title: "IDebugComPlusSymbolProvider2::GetTypesByName | Microsoft Docs"
ms.custom: ""
ms.date: 11/15/2016
ms.prod: "visual-studio-dev14"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "vs-ide-sdk"
ms.tgt_pltfrm: ""
ms.topic: "article"
helpviewer_keywords: 
  - "GetTypesByName"
  - "IDebugComPlusSymbolProvider2::GetTypesByName"
ms.assetid: ef76b1a8-6910-48fe-b4af-d9045eefd23f
caps.latest.revision: 11
ms.author: "gregvanl"
manager: "ghogen"
---
# IDebugComPlusSymbolProvider2::GetTypesByName
[!INCLUDE[vs2017banner](../../../includes/vs2017banner.md)]

Retrieves a type given its name.  
  
## Syntax  
  
```cpp#  
HRESULT GetTypesByName(  
   LPCOLESTR          pszClassName,  
   NAME_MATCH         nameMatch,  
   IEnumDebugFields** ppEnum  
);  
```  
  
```csharp  
int GetTypesByName(  
   string               pszClassName,  
   enum_ NAME_MATCH     nameMatch,  
   out IEnumDebugFields ppEnum  
);  
```  
  
#### Parameters  
 `pszClassName`  
 [in] Name of the type.  
  
 `nameMatch`  
 [in] Selects the type of match, for example, case-sensitive. A value from the [NAME_MATCH](../../../extensibility/debugger/reference/name-match.md) enumeration.  
  
 `ppEnum`  
 [out] An enumerator that contains the type or types with the given name.  
  
## Return Value  
 If successful, returns `S_OK`; otherwise, returns an error code.  
  
## Remarks  
 For generic types, the name to look up for 'List\<int>' or 'List\<int,int>' would be 'List'. If types of the same name appear in multiple modules, the `ppEnum` paramter will contain all copies. You have to use [GetTypeInfo](../../../extensibility/debugger/reference/idebugfield-gettypeinfo.md) and distinguish based on the `guidModule` parameter.  
  
## Example  
 The following example shows how to implement this method for a **CDebugSymbolProvider** object that exposes the [IDebugComPlusSymbolProvider2](../../../extensibility/debugger/reference/idebugcomplussymbolprovider2.md) interface.  
  
```cpp#  
HRESULT CDebugSymbolProvider::GetTypesByName(  
    LPCOLESTR pszClassName,  
    NAME_MATCH nameMatch,  
    IEnumDebugFields** ppEnum  
)  
{  
    HRESULT hr = S_OK;  
    CModIter ModIter;  
    CModule* pmodule; // the iterator owns the reference  
    CFieldList listField;  
  
    ASSERT(IsValidWideStringPtr(pszClassName));  
    ASSERT(IsValidWritePtr(ppEnum, IEnumDebugFields*));  
  
    METHOD_ENTRY( CDebugSymbolProvider::GetTypesByName );  
  
    IfFalseGo( pszClassName && ppEnum, E_INVALIDARG );  
    *ppEnum = NULL;  
  
    IfFailGo( GetModuleIter(&ModIter) );  
  
    hr = S_FALSE;  
  
    if ( nameMatch == nmCaseInsensitive)  
    {  
        while (ModIter.GetNext(&pmodule))  
        {  
            if (pmodule->FindTypesByNameCaseInsensitive( pszClassName,  
                    &listField,  
                    this ) )  
            {  
                hr = S_OK;  
            }  
        }  
    }  
    else  
    {  
        while (ModIter.GetNext(&pmodule))  
        {  
            if (pmodule->FindTypesByName( pszClassName,  
                                          &listField,  
                                          this) )  
            {  
                hr = S_OK;  
            }  
        }  
    }  
  
    // If the list is empty then no type  
    IfFalseGo( listField.GetCount(), E_FAIL );  
  
    // Create enumerator  
    IfFailGo( CreateEnumerator( ppEnum, &listField ) );  
  
Error:  
  
    METHOD_EXIT( CDebugSymbolProvider::GetTypesByName, hr );  
  
    return hr;  
}  
```  
  
## See Also  
 [IDebugComPlusSymbolProvider2](../../../extensibility/debugger/reference/idebugcomplussymbolprovider2.md)
