---
description: "Merge Join Transformation"
title: "Merge Join Transformation | Microsoft Docs"
ms.custom: ""
ms.date: "03/01/2017"
ms.service: sql
ms.reviewer: ""
ms.subservice: integration-services
ms.topic: conceptual
f1_keywords: 
  - "sql13.dts.designer.mergejointrans.f1"
  - "sql13.dts.designer.mergejointransformation.f1"
helpviewer_keywords: 
  - "datasets [Integration Services]"
  - "Merge Join transformation"
  - "datasets [Integration Services], joining"
  - "joining datasets [Integration Services]"
  - "joins [SQL Server], SSIS"
ms.assetid: cd8b0412-f83b-4bd2-b227-e53dcfd941a8
author: chugugrace
ms.author: chugu
---
# Merge Join Transformation

[!INCLUDE[sqlserver-ssis](../../../includes/applies-to-version/sqlserver-ssis.md)]


  The Merge Join transformation provides an output that is generated by joining two sorted datasets using a FULL, LEFT, or INNER join. For example, you can use a LEFT join to join a table that includes product information with a table that lists the country/region in which a product was manufactured. The result is a table that lists all products and their country/region of origin.  
  
 You can configure the Merge Join transformation in the following ways:  
  
-   Specify the join is a FULL, LEFT, or INNER join.  
  
-   Specify the columns the join uses.  
  
-   Specify whether the transformation handles null values as equal to other nulls.  
  
    > [!NOTE]  
    >  If null values are not treated as equal values, the transformation handles null values like the SQL Server Database Engine does.  
  
 This transformation has two inputs and one output. It does not support an error output.  
  
## Input Requirements  
 The Merge Join Transformation requires sorted data for its inputs. For more information about this important requirement, see [Sort Data for the Merge and Merge Join Transformations](../../../integration-services/data-flow/transformations/sort-data-for-the-merge-and-merge-join-transformations.md).  
  
## Join Requirements  
 The Merge Join transformation requires that the joined columns have matching metadata. For example, you cannot join a column that has a numeric data type with a column that has a character data type. If the data has a string data type, the length of the column in the second input must be less than or equal to the length of the column in the first input with which it is merged.  
  
## Buffer Throttling  
 You no longer have to configure the value of the **MaxBuffersPerInput** property because Microsoft has made changes that reduce the risk that the Merge Join transformation will consume excessive memory. This problem sometimes occurred when the multiple inputs of the Merge Join produced data at uneven rates.  
  
## Related Tasks  
 You can set properties through the [!INCLUDE[ssIS](../../../includes/ssis-md.md)] Designer or programmatically.  
  
 For information about how to set properties of this transformation, click one of the following topics:  
  
-   [Extend a Dataset by Using the Merge Join Transformation](../../../integration-services/data-flow/transformations/extend-a-dataset-by-using-the-merge-join-transformation.md)  
  
-   [Set the Properties of a Data Flow Component](../../../integration-services/data-flow/set-the-properties-of-a-data-flow-component.md)  
  
-   [Sort Data for the Merge and Merge Join Transformations](../../../integration-services/data-flow/transformations/sort-data-for-the-merge-and-merge-join-transformations.md)  
  
## Merge Join Transformation Editor
  Use the **Merge Join Transformation Editor** dialog box to specify the join type, the join columns, and the output columns for merging two inputs combined by a join.  
  
> [!IMPORTANT]  
>  The Merge Join Transformation requires sorted data for its inputs. For more information about this important requirement, see [Sort Data for the Merge and Merge Join Transformations](../../../integration-services/data-flow/transformations/sort-data-for-the-merge-and-merge-join-transformations.md).  
  
### Options  
 **Join type**  
 Specify whether you want to use an inner join, left outer join, or full join.  
  
 **Swap Inputs**  
 Switch the order between inputs by using the **Swap Inputs** button. This selection may be useful with the Left outer join option.  
  
 **Input**  
 For each column that you want in the merged output, first select from the list of available inputs.  
  
 Inputs are displayed in two separate tables. Select columns to include in the output. Drag columns to create a join between the tables. To delete a join, select it and then press the DELETE key.  
  
 **Input Column**  
 Select a column to include in the merged output from the list of available columns on the selected input.  
  
 **Output Alias**  
 Type an alias for each output column. The default is the name of the input column; however, you can choose any unique, descriptive name.  
  
## See Also  
 [Merge Transformation](../../../integration-services/data-flow/transformations/merge-transformation.md)   
 [Union All Transformation](../../../integration-services/data-flow/transformations/union-all-transformation.md)   
 [Integration Services Transformations](../../../integration-services/data-flow/transformations/integration-services-transformations.md)  
  
  
