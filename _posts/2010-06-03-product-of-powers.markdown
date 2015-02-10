---
layout: post
title: "PowerProduct"
date: 2010-06-03 09:20:57
categories: source code
tags: wolfram mathematica
---

Wolfram Mathematica package with functions to express the terms of a polynomial like products of powers.

{% highlight mathematica %}
(* ::Package:: *)

(* ::Title:: *)
(*Functions to simplify powers*)


BeginPackage["PowerProduct`"]


GetPowerProductOfNumber::usage = "GetPowerProductOfNumber[number_] returns a number_ 
	expressed as a product of powers."
GetPowerProductOfPolyTerms::usage = "GetPowerProductOfPolyTerms[poly_, x_] returns the 
	polynomial poly_ indicating their terms as products powers."


Begin["`Private`"]


(*
	GetPowerProductOfNumber[number_]

	Returns a number_ expressed as a product of powers.
	
	number_         Integer number greater than 1
*)
GetPowerProductOfNumber[number_] := Module[
	{i, div, temp, pow, first, result},
	
	first = True;

	If[number > 1,	

		(* Get divisors *)
		div = Divisors[number];
	
		(* Delete divisors that are not primes *)
		For[i = 1, i <= Length[div], i++,
			If[!PrimeQ[div[[i]]],
				div = Delete[div, i--],
				Null
			]
		];

		(* Divide the number between each divisor until 1 *)
		temp = number;
		For[i = 1, i <= Length[div], i++,
			pow = 0;
			While[Mod[temp, div[[i]]] == 0,
				temp = temp / div[[i]];
				pow = ++pow
			];
			If[first,
				result = HoldForm[#1^#2]&[div[[i]], pow]; first = False,
				result = HoldForm[#1 * #2^#3]&[result, div[[i]], pow]
			]
		],

		Null
	];

	result
]


(*
	GetPowerProductOfPolyTerms[poly_, x_]

	Returns the polynomial poly_ indicating their terms as products powers.
	
	poly_         Polynomial
	x_            Variable of the polynomial
*)
GetPowerProductOfPolyTerms[poly_, x_] := Module[
	{i, xcoefs, numerator, numeratorpows, denominator, denominatorpows, negative, pows, 
		result},
	
	(* Init result *)
	result = 0;

	(* Get the coefficients of x *)
	xcoefs = CoefficientList[poly, x];

	For[i = 1, i <= Length[xcoefs], i++,

		If[xcoefs[[i]] != 0,

			(* Get the pows of the numerator *)
			numerator = Numerator[xcoefs[[i]]];
			If[numerator < 0,
				negative = True;
				numerator = -numerator,
				negative = False
			];
			If[numerator == 1,
				numeratorpows = 1,
				numeratorpows = GetPowerProductOfNumber[numerator];
			];

			(* Get the pows of the denominator *)
			denominator = Denominator[xcoefs[[i]]];
			If[denominator == 1,
				denominatorpows = 1,
				denominatorpows = GetPowerProductOfNumber[denominator]
			];

			(* Join *)
			If[numerator == denominator,
				pows = HoldForm[#1 * x^#2]&[numerator, i-1],
				pows = HoldForm[(#1/#2) * x^#3]&[numeratorpows, 
					denominatorpows, i-1]
			];
			If[negative,
				result -= pows,
				result += pows
			]
		]
		
	];

	result
]


End[]


EndPackage[]
{% endhighlight %}
