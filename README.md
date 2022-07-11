//@version=4 
study( "[XeL]:trimmedHarrellDavis.weighted.quantileEstimator.outlierAbsDeviations", shorttitle="[XeL]:vw.tHDq.QAD", resolution="", overlay=true, precision=4 ) 


//█████████████████████████████████████████████████████████████████████████████████████
//███████████████████                                             █████████████████████
//███████████████████              QUANTILE  ESTIMATORS           █████████████████████
//███████████████████  Weighted Harrell-Davis Quantile Estimator  █████████████████████
//███████████████████        with Absolute Deviation Fences       █████████████████████
//███████████████████                                             █████████████████████
//█████████████████████████████████████████████████████████████████████████████████████ {
//  Purpose:
//
//    Weighted Quantiles or <<Percentile Ranking>> are quite difficult to find on
//     most systems, also it's non-weighted approach are raraley used to estimate
//     the location parameter of price distribution WHICH IS NOT NORMAL, all this
//     in favour of it's non-robust counterpart, the arithmetic rolling mean or
//     <<Moving Average>> and it's weighted variants like the WMA, VWAP, etc.
//
//       Also, a big drawback from this is that statistics derived from the 
//     Normal-Distribution parameter location (the Mean) deffinitley will not fit 
//     for an efficient nor robust estimation for price distributions, so their
//     moments like standard deviation, kurtosis, skewness, etc. will not should
//     be the better tools to build derived algorithms or technical indicators
//     among price/volume.
//
//       In an effort searching better statistical tools for price distributions,
//    I found the excelent work of Andrey Akinshin that took me to port some of 
//    their Math research contributions for the compute benchmarking field, 
//    and bring it here at the TradingView ecosystem to take a shot at the
//    price distribution crazy fields.  For a better detail of what the Weighted
//    Harrell-Davis Quantile Estimator can do, what a better source than drink
//    directly from the source:
//
//      + The (Trimmed) Weighted Harrell-Davis Quantile estimator; C# & R implementations
//          https://aakinshin.net/posts/trimmed-hdqe/
//
//      + DoubleMAD outlier detector based on the Harrell-Davis quantile estimator:
//          https://aakinshin.net/posts/harrell-davis-double-mad-outlier-detector/
//
//      + Unbiased median absolute deviation based on the Harrell-Davis quantile estimator:
//          https://aakinshin.net/posts/unbiased-mad-hd/
//
//      + Quantile confidence intervals for weighted samples:
//          https://aakinshin.net/posts/weighted-quantiles-ci/
//
//    This indicator contains a DRAFT PROPOSAL from a set of Functions, Methods and
//     Procedures to be used, incorporated forked or modified as part of a Math
//     Functions Library.
//
//  Last revision date:
//
//    8 July 2021
//
//  Licensing:
//
//    This work is licensed under a Attribution-NonCommercial-ShareAlike 4.0 International
//    Copyright (c) 2021 (CC BY-NC-SA 4.0)
//     - https://creativecommons.org/licenses/by-nc-sa/4.0/
//
//  Copyright's & Mentions:
//
//    + Gamma Functions & Beta Probability Density Functions C# implementations by the Math.NET Numerics,
//       part of the Math.NET Project at
//       http://github.com/mathnet/mathnet-numerics : /src/Numerics/SpecialFunctions/Gamma.cs
//                                                  : /src/Numerics/SpecialFunctions/Beta.cs
//
//    + The Regularized Incomplete (Left) Beta Function C# implementation
//       by the SAMTools, htslib project at https://github.com/samtools/htslib/blob/master/kfunc.c
//
//    + The (Trimmed/Windsorized) Weighted Harrell-Davis Quantile estimator; C# & R implementations
//       by Andrey Akinshin at 
//       https://github.com/AndreyAkinshin/perfolizer/blob/master/src/Perfolizer :
//               /Perfolizer/Mathematics/QuantileEstimators/ModifiedHarrellDavisQuantileEstimator.cs
//
//    + External PineScript code, methods, support & consultancy by @PineCoders staff with special mention for:
//       - "ma sorter ('sort by array' example)- JD" by @Duyck at 
//          https://www.tradingview.com/script/Mzc4dmq7-ma-sorter-sort-by-array-example-JD/
//       - While-loop tweak method by @RicardoSantos
//
//    + Porting, mods, compilation and debugging for this script by @XeL_Arjona
//       for the TradingView's @PineCoders community.
//}

//■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■  START OF STUDY  ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■:
//▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼▼{

// ▤▤▤ INCLUDES & DEPENDENCIES :    ▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤
// ▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾{

// ▾▾▾ Precision.CONSTANTS: ▾▾▾ {   The following block of variables brings a set of useful constantants currently not natively
//                           available on PineScript, they try to facilitate global definitions for precision handling as well to
//                           bring auxiliary mathematical support for constant recursion.
//                          DISCLAIMER: This values try to satisfy as close as possible standarization specifications like the
//                           IEEE-754, BUT THEY ARE NOT GUARANTEED OR IN ANY WAY RELATED TO TradingView's OR PineScript OFFICIAL
//                           SUPPORT, IN DIRECT OR INDIRECT WAY.

// Epsilon's (EPS) : { Represents the smallest positive Double value that is greater than zero.
var float   floatPrecision_64bitPositiveEpsilon     =   2.220446049250313e-16    // IEEE 754 64-bit as 2^-52
//var float   floatPrecision_64bitNegativeEpsilon   =   1.1102230246251565e-16   // IEEE 754 64-bit as 2^-53.
var float   floatPrecision_tinyEpsilon              =   5e-324                   // Pine v4 math = 2^-1074 as 2^-1075 returns zero. }

// MaxValue's : { Represents the largest possible value of a number.
//var float floatPrecision_MaxValue                 =   1.7976931348623157e+308  // Largest possible value of a double before isInfinity.
//var int   intPrecision_MaxValue                   =   9223372036854776000      // A greater integer exponent from round( 2^1024 ) output this number.
//var float floatPrecision_powOf2MaxValue           =   8.98846567431158e+307    // Max base 2 possible exponent ^1022 before isInfinity }

// MinValue's : { Represents the smallest possible value of a number.
var float floatPrecision_MinValue                   =   -1.7976931348623157e+308 // Smallest possible value of a double.
//var int   intPrecision_MinValue                   =   -9223372036854776000     // Greater integer exponent than neg.round( 2^1024 ) output this number. }

// GNU's libc Infinity definition from examples : { IEEE 754 floating point numbers can represent positive or negative infinity.
// These values arise from calculations whose result is undefined or cannot be represented accurately.
// You can also deliberately set a floating-point variable to any of them, which is sometimes useful.
var float   floatPrecision_NotANumber               =   na
var float   floatPrecision_isPositiveInfinity       =   1. / nz( 0. )
var float   floatPrecision_isNegativeInfinity       =   log( nz( 0. ) )
var bool    floatPrecision_isInfinity               =   floatPrecision_isPositiveInfinity or floatPrecision_isNegativeInfinity //}

// Loop long definitions:
//var int     loop_Nice                             =   126
//var int     loop_Avg                              =   222
var int     loop_Max                                =   369

// Auxiliary Math.CONSTANTS:
// The log[e](pi) number:
var float               math_LnPi = 1.1447298858494001741434273513530587116472948129153

// The log(2 * sqrt(e / pi)) number:
var float math_LogTwoSqrtEOverPi = 0.6207822376352452223455184457816472122518527279025978

// The number 2 * sqrt(e / pi) number:
var float    math_TwoSqrtEOverPi = 1.8603827342052657173362492472666631120594218414085755

// The order of the Log Gamma aproximation:
var int                    GammaN = 10

// Auxiliary variable when evaluating the Log Gamma function:
var float                  GammaR = 10.900511

// Series of Polynomial coefficients for the Gamma approximation:
var float[]        series_GammaDk = array.from( 2.48574089138753565546e-5,1.05142378581721974210,-3.45687097222016235469,4.51227709466894823700,-2.98285225323576655721,1.05639711577126713077,-1.95428773191645869583e-1,1.70970543404441224307e-2,-5.71926117404305781283e-4,4.63399473359905636708e-6,-2.71994908488607703910e-9 )
        
// Aliases for Global Constants & Definitions: {
NaN         =   floatPrecision_NotANumber
tinyEPS     =   floatPrecision_tinyEpsilon
floatEPS    =   floatPrecision_64bitPositiveEpsilon
floatMin    =   floatPrecision_MinValue
isPosInf    =   floatPrecision_isPositiveInfinity
isNegInf    =   floatPrecision_isNegativeInfinity
isInf       =   floatPrecision_isInfinity
LongLOOP    =   loop_Max                                    //}

// ▴▴▴ EndOf:Precision.CONSTANTS ▴▴▴ }

// ▾▾▾ Array.AUX: ▾▾▾ {  Arrays functions, procedures and/or methods currently not natively available
//               on PineScript, they try to facilitate auxiliary array recurssions.
//              DISCLAIMER: The current set of tools ARE NOT GUARANTEED OR IN ANY WAY RELATED TO TradingView's,
//               and PineScript OFFICIAL SUPPORT IN DIRECT OR INDIRECT WAY.

// Set a (virtual) 2-Dimentional sorted array : {
procedure_arraySet2dSort( toSort, toOrderBySort, sampleSize, ascending ) =>
    // <param> toSort        : Any [float] series that meet >= <param=sampleSize>.
    // <param> toOrderBySort : Any [float] series that meet >= <param=toSort>.
    // <param> sampleSize    : Positive [int] number >= 2 to set array's cluster to sort and order.
    // <param> ascending     : If [bool] is true, sorting is ascending, otherwise descending.
    //
    // <returns>             : A sorted array from <param=toSort> values of <param=sampleSize> lenght.
    //                         An ordered array of <param=toOrderBySort> from the sorted indexing from <param=toSort> with same <param=sampleSize>
    //
    // <copyright>           : Original implementation from "ma sorter ('sort by array' example)- JD" by @Duyck at 
    //                          https://www.tradingview.com/script/Mzc4dmq7-ma-sorter-sort-by-array-example-JD/
    //
    //                         This implementation Mod by @Xel_Arjona for @PineCoders
    //                          2021 CC BY-NC-SA 4.0.
    
    int N = sampleSize
    unsorted  = array.new_float( N, 0 )
    unordered = array.new_float( N, 0 )
    ordered   = array.new_float( N, 0 )
    
    for i = 0 to ( N - 1 )  // Append values to arrays {
        array.set(  unsorted, i, nz(        toSort[ i ] ) )
        array.set( unordered, i, nz( toOrderBySort[ i ] ) ) //}
        
    sorted = array.copy( unsorted )
    array.sort( sorted, ascending ? order.ascending : order.descending )
    
    for j = 0 to ( N - 1 )  // Indexing method for a virtualized 2-dimentional array by @Duyck {
        array.set( ordered, j, nz( array.get( unordered, max( 0, array.indexof( unsorted, array.get( sorted, j ) ) ) ) ) )  //}
    
    [ sorted, ordered ]
    
// EndOf: procedure_arraySet2dSort() }

// ▴▴▴ EndOf: Array.AUX ▴▴▴ }

// ▾▾▾ Math.AUX: ▾▾▾ {  Math functions, procedures and/or methods currently not natively available
//               on PineScript, they try to facilitate auxiliary math recurssions.
//              DISCLAIMER: The current set of tools ARE NOT GUARANTEED OR IN ANY WAY RELATED TO TradingView's,
//               and PineScript OFFICIAL SUPPORT IN DIRECT OR INDIRECT WAY.

// Return square of a value: {
Math_sqr( value ) => value * value
// EndOf: Math_sqr() }

// Array.Math.AUX: {

// Power of array elements: {
array_powOfElements( id, exponent ) =>
    //  <returns> : Power of each iteration value from array <id<
    float [] return_powOfElements = array.new_float( array.size( id ) )
    
    for i = 0 to ( array.size( id ) - 1 )
        array.set( return_powOfElements, i, pow( array.get( id, i ), exponent ) )
    
    return_powOfElements
        
// EndOf: array_powOfElements() }

// EndOf: Array.Math.AUX }

// ▴▴▴ EndOf: Math.AUX ▴▴▴ }

// ▾▾▾ Gamma.Functions.Math.AUX: ▾▾▾ {

// The Logarithm of the Gamma Function: {
function_lnGamma( input ) =>
    // <needs>       : Precision.Constants
    //
    // <param> input : Any float input as argument of the Gamma Function.
    //
    // <returns>     : The logarithm of the Gamma Function.
    //
    // <copyright>   : Ported from the Math.NET Numerics, part of the Math.NET Project
    //                  http://numerics.mathdotnet.com
    //                  http://github.com/mathnet/mathnet-numerics : /src/Numerics/SpecialFunctions/Gamma.cs
    //                  Copyright (c) 2009-2010 Math.NET with MIT/X11 License.
    //
    //                 PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description> : This implementation of the computation of the gamma and logarithm of the gamma function follows the derivation in
    //                  "An Analysis Of The Lanczos Gamma Approximation", Glendon Ralph Pugh, 2004.
    //                  We use the implementation listed on p. 116 which achieves an accuracy of 16 floating point digits. Although 16 digit accuracy
    //                  should be sufficient for double values, improving accuracy is possible (see p. 126 in Pugh).
    //                  Our unit tests suggest that the accuracy of the Gamma function is correct up to 14 floating point digits.
    float z = input
    
    if z < 0.5
    
        float s = array.get( series_GammaDk, 0 )
        for int i = 1 to GammaN
            s += array.get( series_GammaDk, i ) / ( i - z )

        return_lnGamma = math_LnPi - log( sin( math.pi * z ) ) - log( s ) - math_LogTwoSqrtEOverPi - ( ( 0.5 - z ) * log( ( 0.5 - z + GammaR ) / math.e ) )
        
    else
        float s = array.get( series_GammaDk, 0 )
        for int i = 1 to GammaN
            s += array.get( series_GammaDk, i ) / ( z + i - 1.0 )
            
        return_lnGamma = log( s ) + math_LogTwoSqrtEOverPi + ( ( z - 0.5 ) * log( ( z - 0.5 + GammaR ) / math.e ) )

// EndOf: function_lnGamma() }

// The Gamma Function: {
function_Gamma( input ) =>
    // <needs>       : Precision.Constants
    //
    // <param> input : Any float input as argument of the Gamma Function.
    //
    // <returns>     : The Gamma Function.
    //
    // <copyright>   : Ported from the Math.NET Numerics, part of the Math.NET Project
    //                  http://numerics.mathdotnet.com
    //                  http://github.com/mathnet/mathnet-numerics : /src/Numerics/SpecialFunctions/Gamma.cs
    //                  Copyright (c) 2009-2010 Math.NET with MIT/X11 License.
    //
    //                 PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description> : This implementation of the computation of the gamma and logarithm of the gamma function follows the derivation in
    //                  "An Analysis Of The Lanczos Gamma Approximation", Glendon Ralph Pugh, 2004.
    //                  We use the implementation listed on p. 116 which achieves an accuracy of 16 floating point digits. Although 16 digit accuracy
    //                  should be sufficient for double values, improving accuracy is possible (see p. 126 in Pugh).
    //                  Our unit tests suggest that the accuracy of the Gamma function is correct up to 14 floating point digits.
    float z = input

    if z < 0.5

        float s = array.get( series_GammaDk, 0 )
        for int i = 1 to GammaN
            s += array.get( series_GammaDk, i ) / ( i - z )
            
        return_Gamma = math.pi / ( sin( math.pi * z ) ) * s * math_TwoSqrtEOverPi * pow( ( 0.5 - z + GammaR ) / math.e, 0.5 - z )
        
    else
        float s = array.get( series_GammaDk, 0 )
        for int i = 1 to GammaN
            s += array.get( series_GammaDk, i ) / ( z + i - 1.0 )
            
        return_Gamma = s * math_TwoSqrtEOverPi * pow( ( z - 0.5 + GammaR ) / math.e, z - 0.5 )

// EndOf: function_Gamma() }

// ▴▴▴ EndOf: Gamma.Functions.Math.AUX ▴▴▴ }

// ▾▾▾ Beta.ContinuousDistributions.Math.AUX: ▾▾▾ {

// [Group] The Regularized Incomplete -Left- Beta: {

// The Regularized Incomplete -Left- Beta Continued Fraction Procedure: {
procedure_regIncBetaCF( alpha, beta, input ) =>
    // <needs>          : Precision.Constants
    //                    function_lnGamma()
    //
    // <param> alpha    : A positive float number as the first Beta shape parameter.
    // <param> beta     : A positive float number as the second Beta shape parameter.
    // <param> integral : The upper limit of the integral parameter.
    //
    // <returns>        : Returns a Continued Fraction method for the Regularized Lower Incomplete Beta function caller.
    //
    // <copyright>      : Ported from the SAMTools, htslib project:
    //                      https://github.com/samtools/htslib/blob/develop/kfunc.c
    //                      Copyright (C) 2010, 2013-2014, 2020 Genome Research Ltd.
    //                      Copyright (C) 2011 Attractive Chaos <attractor@live.co.uk> with MIT/X11 License.
    //
    //                    PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description>    : I_x(a,b) = 1/Beta(a,b) * int(t^(a-1)*(1-t)^(b-1),t=0..x) for real a &gt; 0, b &gt; 0, 1 &gt;= x &gt;= 0.
    //                      Regularized incomplete beta function. The method is taken from
    //                      Numerical Recipe in C, 2nd edition, section 6.4.
    //                      betai(a,b,x) * gamma(a) * gamma(b) / gamma(a+b):
    //

    var float GAMMA_EPS = floatEPS,    var float TINY = tinyEPS
    float x = input,        float a = alpha,             float b = beta
    float lnGamma_a     =   function_lnGamma( a )
    float lnGamma_b     =   function_lnGamma( b )
    float lnGamma_ab    =   function_lnGamma( a + b )

    if a < 0.0 or b < 0.0
        return_regIncBeta = NaN

    if x < 0.0 or x > 1.0
        return_regIncBeta = NaN
    
    float bt = ( x == 0.0 or x == 1.0 ) ? 0.0 : exp( lnGamma_ab - lnGamma_a - lnGamma_b + ( a * log( x ) ) + ( b * log( 1.0 - x ) ) )

	float f = 1.,    float C = f,      float D = 0.
	// Modified Lentz's algorithm for computing continued fraction
	for j = 1 to LongLOOP
		float aa = NaN,     float d = NaN
		int m = j / 2
		if j == 0
		    aa := 1.0
        else if j % 2 == 0
            aa := m * ( b - m ) * x / ( ( a + 2.0 * m - 1.0 ) * ( a + 2.0 * m ) )
        else
		    aa := - ( a + m ) * ( a + b + m ) * x / ( ( a + 2.0 * m ) * ( a + 2.0 * m + 1.0 ) )
		
		D := 1.0 + aa * D
		if D < TINY
		    D := TINY
		    
		C := 1.0 + aa / C
		if C < TINY
		    C := TINY
		    
		D := 1.0 / D
		d := C * D
		f *= d
		
		if abs( d - 1.0 ) < GAMMA_EPS
		    break
    
	return_regIncBeta = bt / a / f
	
// EndOf: procedure_regIncBetaCF() }

// The Regularized Incomplete -Left- Beta Function caller: {
function_regIncBeta( alpha, beta, input ) =>
    // <needs>          : procedure_regIncBetaCF()
    //
    // <param> alpha    : A positive float number as the first Beta shape parameter.
    // <param> beta     : A positive float number as the second Beta shape parameter.
    // <param> integral : The upper limit of the integral parameter.
    //
    // <returns>        : Returns the Regularized Lower Incomplete Beta function.
    //
    // <copyright>      : Ported from the SAMTools, htslib project:
    //                      https://github.com/samtools/htslib/blob/develop/kfunc.c
    //                      Copyright (C) 2010, 2013-2014, 2020 Genome Research Ltd.
    //                      Copyright (C) 2011 Attractive Chaos <attractor@live.co.uk> with MIT/X11 License.
    //
    //                    PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description>    : I_x(a,b) = 1/Beta(a,b) * int(t^(a-1)*(1-t)^(b-1),t=0..x) for real a &gt; 0, b &gt; 0, 1 &gt;= x &gt;= 0.
    //                      Regularized incomplete beta function. The method is taken from
    //                      Numerical Recipe in C, 2nd edition, section 6.4.
    //                      betai(a,b,x) * gamma(a) * gamma(b) / gamma(a+b):
    //
    
    float x = input,        float a = alpha,        float b = beta

	return_regIncBeta = x < ( a + 1.0 ) / ( a + b + 2.0 ) ? procedure_regIncBetaCF( a, b, x ) : 1.0 - procedure_regIncBetaCF( b, a, 1.0 - x )
	
// EndOf: regIncBeta() }

// EndOf[Group]: The Regularized Incomplete -Left- Beta }

// The Logarithm Probability Density of the Beta distribution (lnPDF): {
function_betaPdfLn( alpha, beta, location ) =>
    // <needs>          : Precision.Constants
    //                    function_lnGamma()
    //
    // <param> alpha    : A positive float number as the 1st. Beta shape parameter with range: α ≥ 0.
    // <param> beta     : A positive float number as the 2nd. Beta shape parameter with range: β ≥ 0.
    // <param> location : The location at which to compute the density.
    //
    // <returns>        : The Log(e) density at <param=location>.
    //
    // <copyright>      : Ported from the Math.NET Numerics, part of the Math.NET Project
    //                      http://numerics.mathdotnet.com
    //                      http://github.com/mathnet/mathnet-numerics : /src/Numerics/SpecialFunctions/Beta.cs
    //                      Copyright (c) 2009-2010 Math.NET with MIT/X11 License.
    //
    //                    PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description>    : The probability density function (pdf) of the beta distribution, for 0 ≤ x ≤ 1,
    //                      and shape parameters α, β > 0, is a power function of the variable x and of
    //                      its reflection (1 − x).
    //

    float a = alpha,    float b = beta,     float x = location

    if a < 0.0 or b < 0.0
        return_betaPdfLn = NaN

    if x < 0.0 or x > 1.0
        return_betaPdfLn = isNegInf

    if a == isPosInf and b == isPosInf
        return_betaPdfLn = x == 0.5 ? isPosInf : isNegInf
 
    if a == isPosInf
        return_betaPdfLn = x == 1.0 ? isPosInf : isNegInf
 
    if b == isPosInf
        return_betaPdfLn = x == 0.0 ? isPosInf : isNegInf

    if a == 0.0 and b == 0.0
        return_betaPdfLn = x == 0.0 or x == 1.0 ? isPosInf : isNegInf

    if a == 0.0
        return_betaPdfLn = x == 0.0 ? isPosInf : isNegInf

    if b == 0.0
        return_betaPdfLn = x == 1.0 ? isPosInf : isNegInf

    if a == 1.0 and b == 1.0
        return_betaPdfLn = 0.0

    var aa = function_lnGamma( a + b ) - function_lnGamma( a ) - function_lnGamma( b )
    var bb = x == 0.0 ? ( a == 1.0 ? 0.0 : isNegInf ) : ( a - 1.0 ) * log( x )
    var cc = x == 1.0 ? ( b == 1.0 ? 0.0 : isNegInf ) : ( b - 1.0 ) * log( 1.0 - x )

    return_betaPdfLn = aa + bb + cc

// EndOf: function_betaPdfLn() }

// The Probability Density of the Beta distribution (PDF): {
function_betaPdf( alpha, beta, location ) =>
    // <needs>          : Precision.Constants
    //                    function_Gamma()
    //                    function_betaPdfLn()
    //
    // <param> alpha    : A positive float number as the 1st. Beta shape parameter with range: α ≥ 0.
    // <param> beta     : A positive float number as the 2nd. Beta shape parameter with range: β ≥ 0.
    // <param> location : The location at which to compute the density.
    //
    // <returns>        : The Beta density at <param=location>.
    //
    // <copyright>      : Ported from the Math.NET Numerics, part of the Math.NET Project
    //                      http://numerics.mathdotnet.com
    //                      http://github.com/mathnet/mathnet-numerics : /src/Numerics/SpecialFunctions/Beta.cs
    //                      Copyright (c) 2009-2010 Math.NET with MIT/X11 License.
    //
    //                    PineScript port by @Xel_Arjona for @PineCoders
    //                      2021 CC BY-NC-SA 4.0.
    //
    // <description>    : The probability density function (pdf) of the beta distribution, for 0 ≤ x ≤ 1,
    //                      and shape parameters α, β > 0, is a power function of the variable x and of
    //                      its reflection (1 − x).
    //

    float a = alpha,    float b = beta,     float x = location

    if a < 0.0 or b < 0.0
        return_betaPdf = NaN

    if x < 0.0 or x > 1.0
        return_betaPdf = 0.0

    if a == isPosInf and b == isPosInf
        return_betaPdf = x == 0.5 ? isPosInf : 0.0
 
    if a == isPosInf
        return_betaPdf = x == 1.0 ? isPosInf : 0.0
 
    if b == isPosInf
        return_betaPdf = x == 0.0 ? isPosInf : 0.0

    if a == 0.0 and b == 0.0
        return_betaPdf = x == 0.0 or x == 1.0 ? isPosInf : 0.0

    if a == 0.0
        return_betaPdf = x == 0.0 ? isPosInf : 0.0

    if b == 0.0
        return_betaPdf = x == 1.0 ? isPosInf : 0.0

    if a == 1.0 and b == 1.0
        return_betaPdfLn = 1.0

    if a > 80.0 and b > 80.0
        return_betaPdfLn = exp( function_betaPdfLn( a, b, x ) )

    var bb = function_Gamma( a + b ) / ( function_Gamma( a ) * function_Gamma( b ) )
    
    return_betaPdf = bb * pow( x, a - 1.0 ) * pow( 1.0 - x, b - 1.0 )
    
// EndOf: function_betaPdf()

//--- END OF DEPENDENCY AND INCLUDE PROCEDURES --- }

// ▴▴▴ EndOf: Beta.ContinuousDistributions.Math.AUX ▴▴▴ }

// ▾▾▾ QuantileEstimators.Statistics.Math.AUX: ▾▾▾ {

// The Trimmed Weighted Harrell-Davis Quantile Estimator : {
function_twHDquantile( sampleValues, sampleWeights, sampleSize, probability, trimPercent, calcSecondMoment ) =>
    // <needs>          : Precision.Constants
    //                    procedure_arraySet2dSort
    //                    function_regIncBeta()
    //                    function_betaPdf()
    //
    // <param> sampleValues  : Original sample assuming that it’s always contain sorted real [float] numbers.
    // <param> sampleWeights : A vector of weights ordered real [float] numbers that has the same length as <param=sampleValues>.
    // <param> sampleSize    : The [int] number of <bar_index> elements to build all samples.
    // <param> probability   : Estimation of the p-th % quantile based on <param=sampleValues> within .0 < probability < 1. range.
    // <param> trimPercent   : Cuting factor from what to trimm off calculations from the lowest-density BetaCDF weighting function.
    //
    // <returns>             : The qth-Quantile nearest number from a partition given by <param=probability> estimation that
    //                          satisfies the k-th element of a distribution of <param=sampleValues> weighted within
    //                          <param=sampleWeighted> input.
    //
    // <copyright>      : Original Estimator implementation by Harrell, F.E. and Davis, C.E., 1982.
    //                      - A new distribution-free quantile estimator. Biometrika, 69(3), pp.635-640.
    //                    Trimmed Weighted Harrell-Davis implementation by Andrey Akinshin, CC BY-NC-SA 4.0.
    //                      - https://aakinshin.net/posts/trimmed-hdqe/
    //                    PineScript implementation by @Xel_Arjona for @PineCoders
    //                      - 2021 CC BY-NC-SA 4.0.
    //
    // <description>    : From Wikipedia: --In statistics and probability, quantiles are cut points dividing the range of a 
    //                      probability distribution into continuous intervals with equal probabilities, or dividing the
    //                      observations in a sample in the same way.--
    //                        Weighted quantiles are rareley implemented but the author of the original implementation from this
    //                      PineScript port argues that his "trimmed" modification is more efficient that tradditional 
    //                      linear-interpolation methods.
    //
    // <references>     : - Trimmed modification of the Harrell-Davis quantile estimator:
    //                        https://aakinshin.net/posts/trimmed-hdqe/
    //                    - Trimmed or Truncated (mean) Wikipedia definition:
    //                        https://en.wikipedia.org/wiki/Truncated_mean
    //                    - Harrell, F.E. and Davis, C.E., 1982. A new distribution-free quantile estimator. Biometrika, 69(3), pp.635-640.
    //                        https://pdfs.semanticscholar.org/1a48/9bb74293753023c5bb6bff8e41e8fe68060f.pdf
    //

    // Sample process: Sort values and order indexing of weights for the gieven size:
    [ sortedVals_a, orderedWeights_a ] = procedure_arraySet2dSort( sampleValues, sampleWeights, sampleSize, true )
    
    // Define parameters for Beta weighting functions:
    int N = array.size( orderedWeights_a ),         bool csm = calcSecondMoment
    float P  = ( probability == 100 ?   100. - floatMin :
                 probability == 0.0 ?   0.0  + floatEPS : probability )
    
    sqrOrderedWeights_a =   array_powOfElements( orderedWeights_a, 2 )
    totalWeight         =   array.sum( orderedWeights_a )
    totalSqrWeight      =   array.sum( sqrOrderedWeights_a )
    wN                  =   Math_sqr( totalWeight ) / totalSqrWeight    // Kish's effective sampleWeight size counter
    float targetPercent = 1.0 - trimPercent                             // Trim Value
    bool  symmetricMode = abs( P - 0.5 ) < 1e-9                         // Assertion to split probability in half
    float  a = ( wN + 1 ) * P,  float  b = ( wN + 1 ) * ( 1 - P )       // The shape of α and β for the betaCDF()
    float c1 = 0,               float c2 = csm ? 0 : NaN                // Initialize Moment's variables
    
    // Preparation:
    float betaCdfLeft = NaN,    float betaCdfRight = 0,     float betaCdfNext = 0
    float betaPdfLeft = 0,      float betaPdfRight = 0,     float betaPdfNext = 0
    float [] W_a      = array.new_float( N )                            // Array of Weights

    int   indexLeft     = 0,    float elementProbs = 0
    float probabilityL  = NaN,  float probabilityR = 0
    for indexL = 0 to ( N - 1 )
        elementProbs := array.get( orderedWeights_a, indexL ) / totalWeight
        probabilityL := nz( probabilityR )
        probabilityR := probabilityL + elementProbs
        indexLeft    := indexL
        if ( probabilityR >= P or indexLeft == N - 1 )
            break
    
    int indexRight   =  indexLeft
    betaCdfLeft     :=  function_regIncBeta( a, b, probabilityL )
    betaCdfRight    :=  function_regIncBeta( a, b, probabilityR )
    // Process Left
    array.set( W_a, indexLeft, betaCdfRight - betaCdfLeft )
    c1 += array.get( W_a, indexLeft ) * array.get( sortedVals_a, indexLeft )
    c2 += csm ? array.get( W_a, indexLeft ) * pow( array.get( sortedVals_a, indexLeft ), 2 ) : NaN
    // Set BetaPDF values for assertions:
    betaPdfLeft  := function_betaPdf( probabilityL, a, b )
    betaPdfRight := function_betaPdf( probabilityR, a, b )
    
    int indexNext = 0,    float probabilityN = 0
    for while = 0 to N      // Let's emulate a while-loop, method courtesy from @RicardoSantos
        if ( ( betaCdfRight - betaCdfLeft < targetPercent or symmetricMode and ( indexLeft != N - 1 - indexRight ) ) and ( indexLeft > 0 or indexRight < N - 1 ) )

            if ( indexLeft > 0 and ( betaPdfLeft > betaPdfRight or indexRight == N - 1 ) )
                // Expand to the left
                indexNext    := indexLeft - 1
                probabilityN := probabilityL - ( array.get( orderedWeights_a, indexNext ) / totalWeight )
                betaCdfNext  := function_regIncBeta( a, b, probabilityN )
                betaPdfNext  := function_betaPdf( probabilityN, a, b )
                // Process Next to the Left
                array.set( W_a, indexNext, betaCdfLeft - betaCdfNext )
                c1 += array.get( W_a, indexNext ) * array.get( sortedVals_a, indexNext )
                c2 += csm ? array.get( W_a, indexNext ) * pow( array.get( sortedVals_a, indexNext ), 2 ) : NaN
                // Reset Left values
                indexLeft    := indexNext
                probabilityL := probabilityN
                betaCdfLeft  := betaCdfNext
                betaPdfLeft  := betaPdfNext
                
            else
                // Expand to rhe right
                indexNext    := indexRight + 1
                probabilityN := probabilityR + ( array.get( orderedWeights_a, indexNext ) / totalWeight )
                betaCdfNext  := function_regIncBeta( a, b, probabilityN )
                betaPdfNext  := function_betaPdf( probabilityN, a, b )
                // Process Next to the Right
                array.set( W_a, indexNext, betaCdfNext - betaCdfRight )
                c1 += array.get( W_a, indexNext ) * array.get( sortedVals_a, indexNext )
                c2 += csm ? array.get( W_a, indexNext ) * pow( array.get( sortedVals_a, indexNext ), 2 ) : NaN
                // Reset Left values
                indexRight   := indexNext
                probabilityR := probabilityN
                betaCdfRight := betaCdfNext
                betaPdfRight := betaPdfNext
    
    // Process Trimmed Factor
    scaleFactor = 1 / ( betaCdfRight - betaCdfLeft )
    c1 *= scaleFactor
    c2 *= csm ? scaleFactor : NaN

    // Define return moments in tuples
    return_c1 = c1,     return_c2 = c2
    [ return_c1,        return_c2 ]
    
// EndOf: twHDquantile() }

// ▴▴▴ EndOf: Quantiles.Estimators.Statistics.Math.AUX ▴▴▴ }

// ▾▾▾ OutlierDetection.Statistics.Math.AUX: ▾▾▾ {

// The Trimmed Weighted Harrell-Davis Quantile Aboluste Deviation: {
function_twHDquantileAbsDev( tHDqEstimator, sampleValues, sampleWeights, sampleSize, trimPercent, deviations, doubleQad ) =>
    // <needs>                : function_twHDquantile()
    //                        : median() from pineScript =< v.4
    //
    // <param> tHDqEstimator  : Any processed quantile estimate moment from 'function_twHDquantile()' related to sampleValues.
    // <param> sampleValues   : Same vector of real [float] numbers used for <param=tHDqEstimator>.
    // <param> sampleWeights  : Same vector of real [float] weights numbers used for <param=tHDqEstimator>.
    // <param> sampleSize     : The [int] number of <bar_index> elements to build all samples.
    // <param> trimPercent    : Any real [float] factor number that meets 0.0 <= trimPercent <= .5
    // <param> deviations     : Any real [float] number that multiply k-th., times the QAD calculation at up-down fences.
    // <param> doubleQad      : Boolean assertion: If 'true' will compute separete deviations for each outlier fences.
    //
    // <returns>              : Both the Upper and Lower fences of a calculated (Double) Quantile Absolute Deviation
    //                          from within all input parameters <param>.
    //
    // <copyright>      : Original Estimator implementation by Harrell, F.E. and Davis, C.E., 1982.
    //                      - A new distribution-free quantile estimator. Biometrika, 69(3), pp.635-640.
    //                    DoubleMAD outlier detector based on the Harrell-Davis quantile estimator implementation by Andrey Akinshin, CC BY-NC-SA 4.0.
    //                      - https://aakinshin.net/posts/harrell-davis-double-mad-outlier-detector/
    //                    Rosenmai, Peter. “Using the Median Absolute Deviation to Find Outliers.” Eureka Statistics. November 25, 2013. Accessed June 22, 2020.
    //                      - https://eurekastatistics.com/using-the-median-absolute-deviation-to-find-outliers/
    //                    PineScript implementation by @Xel_Arjona for @PineCoders
    //                      - 2021 CC BY-NC-SA 4.0.
    //
    // <description>    : From Akinshin's blog: --Outlier detection is an important step in data processing. Unfortunately,
    //                      if the distribution is not normal (e.g., right-skewed and heavy-tailed), it’s hard to choose a 
    //                      robust outlier detection algorithm that will not be affected by tricky distribution properties. 
    //                      During the last several years, I tried many different approaches, but I was not satisfied with
    //                      their results. Finally, I found an algorithm to which I have (almost) no complaints.
    //                      It’s based on the double median absolute deviation and the Harrell-Davis quantile estimator.--
    //
    // <references>     : - https://aakinshin.net/posts/harrell-davis-double-mad-outlier-detector/
    //                    - Harrell, F.E. and Davis, C.E., 1982. A new distribution-free quantile estimator. Biometrika, 69(3), pp.635-640.
    //                        https://pdfs.semanticscholar.org/1a48/9bb74293753023c5bb6bff8e41e8fe68060f.pdf
    
    // Function variable initializations:
    int N   = sampleSize,   float k = deviations,       bool dq = doubleQad
    var float consistencyConstant = 1.482602218505602
    float return_UpperFence = NaN,      float return_LowerFence = NaN
    
    // Array initializations:
    float[] UpDeviations_a  = array.new_float( N )
    float[] LoDeviations_a  = array.new_float( N )
    float[] deviations_a    = array.new_float( N )
    
    // Get a processed trimmed weighted Harrell-Davis quantile estimator:
    Qp = tHDqEstimator

    // Process Deviations method:    
    float maxVal    = highest( 1 ),     float minVal = lowest( 1 )
    
    for i = 0 to ( N - 1 )
        if ( dq == true )
            if maxVal[ i ] >= Qp
                array.set( UpDeviations_a, i, abs( maxVal[ i ] - Qp ) )
            if minVal[ i ] <= Qp
                array.set( LoDeviations_a, i, abs( minVal[ i ] - Qp ) )
        else
            array.set( deviations_a, i, abs( sampleValues[ i ] - Qp ) )

    // Process Quantile Absolute Deviations [Use faster array.median() if <param=sampleWeights> defined to ones]
    float maxWeight = maxVal >= Qp ? sampleWeights : 1.0    // Replace maxWeight NaN's with ones for rejected max assertion.
    float minWeight = minVal <= Qp ? sampleWeights : 1.0    // Replace minWeight NaN's with ones for rejected min assertion.

    if ( dq == true )               // Double Quantile Absolute Deviation:
        if ( sampleWeights == 1 )   // Use faster 'array.median()' if ones defined at <param=sampleWeights>.
            Qad                 =    array.median( UpDeviations_a )
            loQad               =    array.median( LoDeviations_a )
            return_UpperFence  :=    Qp + k * ( Qad   * consistencyConstant )
            return_LowerFence  :=    Qp - k * ( loQad * consistencyConstant )

        [ Qad,   Qad2   ]   =    function_twHDquantile( array.get( UpDeviations_a, 0 ), maxWeight, N, .50, trimPercent, false )
        [ loQad, loQad2 ]   =    function_twHDquantile( array.get( LoDeviations_a, 0 ), minWeight, N, .50, trimPercent, false )
        return_UpperFence  :=    Qp + k * ( Qad   * consistencyConstant )
        return_LowerFence  :=    Qp - k * ( loQad * consistencyConstant )
    
    else                            // Quantile Absolute Deviation:
        if ( sampleWeights == 1 )   // Use faster 'array.median()' if ones defined at <param=sampleWeights>.
            Qad                =    array.median( deviations_a )
            return_UpperFence :=    Qp + k * ( Qad * consistencyConstant )
            return_LowerFence :=    Qp - k * ( Qad * consistencyConstant )
        
        [ Qad, Qad2 ]      =    function_twHDquantile( array.get( deviations_a, 0 ), sampleWeights, N, .50, trimPercent, false )
        return_UpperFence :=    Qp + k * ( Qad * consistencyConstant )
        return_LowerFence :=    Qp - k * ( Qad * consistencyConstant )
        
    [ return_LowerFence, return_UpperFence ]
// EndOf: function_twHDquantileAbsDev() }

// ▴▴▴ EndOf: OutlierDetection.Statistics.Math.AUX ▴▴▴ }

// ▴▴▴ EndOf: INCLUDES & DEPENDENCIES ▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴}

// ▤▤▤ PARAMETERS & SETTINGS :      ▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤
// ▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾{

// [ QUANTILE ESTIMATION ] Group
tg_vw   =   input( false,   title = "Override Volume Weighting [Faster]:",              group = "[ QUANTILE ESTIMATION ]", tooltip = "As no Volume-weight's defined [ SampleWeights == ones ], TradingView's internal 'array.median()' is used only at Quantile Absolute Deviation fences process for better speed compilation, but 'function_twHDquantile()' still used as the main quantile estimator & deviator." )
in_x    =   input( hl2,     title = "Sample Source:",                                   group = "[ QUANTILE ESTIMATION ]", tooltip = "Input [series] for Quantile Estimation [sampleValues]" )
in_N    =   input( 39,      title = "Sample Size:",                                     group = "[ QUANTILE ESTIMATION ]", tooltip = "Rolling bars-back from within [bar_index] to compute Quantile & QAD's on selected TimeFrame." )
in_prob =   input( 50.0,    title = "Probability [ 50 % is Median ]:",                  minval=0.0, maxval=100, group="[ QUANTILE ESTIMATION ]", tooltip="0 % is lower or minimum probability where 100 % maximum. 50 % by definition is the Estimated Median [Default parameter]." )
// [ QUANTILE ABOLUTE DEVIATION FENCES ] Group
tg_dqad =   input( true,    title = "Use DoubleQAD Fence outlier detection:",           group="[ QUANTILE ABOLUTE DEVIATION FENCES ]", tooltip="DoubleQAD consumes more compilation time as it uses additional order-statistical recursion, but gives a more realistic outlier detection for a non-parametric distribution like price series tend to have, as they'r Not-normal or symetrical." )
in_k    =   input( 1.213,   title = "Absolute Deviation 'k' multiplier:",               minval=.01, maxval=4.0, group="[ QUANTILE ABOLUTE DEVIATION FENCES ]", tooltip="When not in DoubleQAD, this multiple tend to make a better estimate approximation fit of a Normal-Distribution sigma effect behaviour [ 1x~68%, 2x~95%, 3x~99.7% ]." )
// [ GLOBAL PARAMS ] Group
in_trim =   input( 9.0 ,    title = "Trim % cut-off from BetaCDF weighting function:",  minval=0.0, maxval=50.0, group="[ GLOBAL PARAMETER ]", tooltip="In a percent factor, it cut's out how much from the lower-density of the Weighting Beta[Cummulative Density Function] will be trimmed off replacing sample calculations with NaN's, this reduce computation time and make a more robust estimation against big outliers from the sampleValues. [ 0% use the full BetaCDF weights in sample ]." )

// ▴▴▴ EndOf: PARAMETERS & SETTINGS ▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴}

// ▤▤▤ ALGORITHM DEFINITIONS :      ▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤
// ▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾{

// Define Weighting [series]: If volume override, feed each iteration with ones as an equal weighting factor: {
weight = tg_vw ? 1 : nz( volume, 10 ) //}

// Convert percentage input numbers to it's factorial function <parameter> definition: {
prob_pctToFact = in_prob / 100
trim_pctToFact = in_trim / 100 // }

// The Weighted Harrell-Davis Estimates Confidence Intervals using the Maritz-Jarrett method: {
[ hdM1, hdM2 ] = function_twHDquantile( in_x, weight, in_N, prob_pctToFact, trim_pctToFact, true )
wHDqp       =   hdM1                                // Quantile Probability Estimation (First Moment).
wHDse       =   sqrt( hdM2 - ( hdM1 * hdM1 ) )      // The Standard Error estimation. }

// Quantile Absolute Deviation fences bands: {
[ loBand, upBand ] = function_twHDquantileAbsDev( wHDqp, in_x, weight, in_N, trim_pctToFact, in_k, tg_dqad ) //}

// ▴▴▴ EndOf: ALGORITHM DEFINITIONS ▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴}

// ▤▤▤ PLOTS :                      ▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤▤
// ▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾▾{

ub = plot( upBand,  title = "twHDqad.UpperFence",                                       color = color.rgb( 0, 50, 255, 50 )  )
Qp = plot(  wHDqp,  title = "Volume Weighted Trimmed.HarrellDavis.Quantile.Estimator",  color = color.rgb( 50, 150, 255, 0 ) )
lb = plot( loBand,  title = "twHDqad.LowerFence",                                       color = color.rgb( 255, 0, 0, 50 )   )

fill( Qp, lb,       title = "Lower Deviation Area",                                     color = color.rgb( 255,0, 0, 81 ) )
fill( ub, Qp,       title = "Upper Deviation Area",                                     color = color.rgb( 0, 50, 255, 81 ) )

// ▴▴▴ EndOf: Plots ▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴▴}

// ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
// ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■   END OF STUDY  ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ }
