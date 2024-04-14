<strong>
Title: Top 10 Franchises Analysis </strong>

Description:
This SQL code analyzes the top 10 franchises based on female CNIC activations during odd hours. It retrieves data from the 'tbl_total_BVS_activity' table in the specified database and calculates the total activations and female CNIC activations for each franchise. The analysis is performed for the months of January 2024 and the previous six months. Franchises with more than 49 total activations in January 2024 are considered.

Purpose:
The purpose of this analysis is to identify the top-performing franchises in terms of female CNIC activations during odd hours. This information can be used for performance evaluation and targeted marketing strategies.

*/

-- Common Table Expressions (CTEs)
WITH MonthlyActivations AS (
    -- Calculate total activations and female CNIC activations for each franchise
    SELECT 
        FRANCHISE_ID,
        MNTH,
        COUNT(*) AS TotalActivations,
        SUM(CASE WHEN gender_type = 'Female' THEN 1 ELSE 0 END) AS FemaleCNIC
    FROM 
        [YourDatabaseName].[dbo].[tbl_total_BVS_activityabc]  -- Replace 'YourDatabaseName' with your actual database name
    WHERE 
        timeslab = '10PM TO 8AM'  -- Filtering odd hours
       AND MNTH IN ('1/1/2024', '12/1/2023', '11/1/2023', '10/1/2023', '9/1/2023', '8/1/2023', '7/1/2023')
       AND [Activity_Type] IN ('Biometric Re-Verification', 'Biometric Verification', 'New Sale', 'Re-connect Sale')
        AND [Biometric_Status] = 'SUCCESS'
    GROUP BY 
        FRANCHISE_ID, MNTH
),
TopUsersJan24 AS (
    -- Select top 10 franchises with more than 49 total activations in January 2024
    SELECT TOP 10
        FRANCHISE_ID,
        FemaleCNIC,
        TotalActivations
    FROM 
        MonthlyActivations
    WHERE 
        MNTH = '1/1/2024'
    GROUP BY 
        FRANCHISE_ID, FemaleCNIC, TotalActivations
    HAVING 
        SUM(TotalActivations) > 49  -- Total Activations>49
    ORDER BY 
        TotalActivations DESC
)
-- Main query to compare activations for top franchises across different months
SELECT 
    T.FRANCHISE_ID,
    T.FemaleCNIC AS Jan_24_Female,
    T.TotalActivations AS Jan_24_Activations,
    ISNULL(D.FemaleCNIC, 0) AS Dec_23_Female,
    ISNULL(D.TotalActivations, 0) AS Dec_23_Activations,
    ISNULL(N.FemaleCNIC, 0) AS Nov_23_Female,
    ISNULL(N.TotalActivations, 0) AS Nov_23_Activations,
    ISNULL(O.FemaleCNIC, 0) AS Oct_23_Female,
    ISNULL(O.TotalActivations, 0) AS Oct_23_Activations,
    ISNULL(S.FemaleCNIC, 0) AS Sep_23_Female,
    ISNULL(S.TotalActivations, 0) AS Sep_23_Activations,
    ISNULL(A.FemaleCNIC, 0) AS Aug_23_Female,
    ISNULL(A.TotalActivations, 0) AS Aug_23_Activations,
    ISNULL(J.FemaleCNIC, 0) AS Jul_23_Female,
    ISNULL(J.TotalActivations, 0) AS Jul_23_Activations
INTO tblT10femaleOddhrs  -- Storing results into a table
FROM 
    TopUsersJan24 T
LEFT JOIN 
    MonthlyActivations D ON T.FRANCHISE_ID = D.FRANCHISE_ID AND D.MNTH = '12/1/2023'
LEFT JOIN 
    MonthlyActivations N ON T.FRANCHISE_ID = N.FRANCHISE_ID AND N.MNTH = '11/1/2023'
LEFT JOIN 
    MonthlyActivations O ON T.FRANCHISE_ID = O.FRANCHISE_ID AND O.MNTH = '10/1/2023'
LEFT JOIN 
    MonthlyActivations S ON T.FRANCHISE_ID = S.FRANCHISE_ID AND S.MNTH = '9/1/2023'
LEFT JOIN 
    MonthlyActivations A ON T.FRANCHISE_ID = A.FRANCHISE_ID AND A.MNTH = '8/1/2023'
LEFT JOIN 
    MonthlyActivations J ON T.FRANCHISE_ID = J.FRANCHISE_ID AND J.MNTH = '7/1/2023';
