# Google_Sheet_Programmatic_Thinking
Before learning to code I got pretty good at making very complicated Google Sheets - here's an example with a particularly interesting problem.

https://docs.google.com/spreadsheets/d/14Fc5vvPZVEtUMoejkr47A_Q8nLbjUHbQgU0ejzBEsFk/edit#gid=0

This is a largely simple sheet for a small business to log and track orders. I added some basic ArrayFormulas to automate some of the tasks they were doing manually. Columns N:S are examples of calculations they were doing everyday, but are now automated.

Column N is a bit more interesting:
Array_constrain(ArrayFormula(iferror(vlookup(G4:G, Costs!$C$1:$E, 3, False)*(if(isblank(H4:H), 1, H4:H)), "")),counta(B4:B)+1, 1)

This is a vlookup to another cell to find the cost of the postage and times it by the post quantity. There is an if statement to set post quantity to 1 if left blank so the user doesn't have to do unnesecarry formulas for the formula. Similarly G4:G (postage type) is a dropdown based on the same sheet as this vlookup, so in the future if postage costs change either name or cost they only need to update their costs in one place.

So far, so standard. However, the reason I've included this sheet is because the business allows customers to order in Metres or Feet and Inches, but their costs are calculated in Metres Square. The users had been manually converting everything using google.

I've written three formulas to automate this:

Cell AE4 if for Width:
=array_constrain(ArrayFormula(ifs(
isblank(V4:V), if(
not(isblank(Y4:Y)), 
((VALUE(REGEXEXTRACT(text(Y4:Y, "0.00"), "[0-9.]+"))
+(VALUE(REGEXEXTRACT(text(Z4:Z, "0.00"), "[0-9.]+"))/12))
*0.3048), ""),
 
lower(REGEXEXTRACT(V4:V, "[A-Za-z]+")) = "m", VALUE(REGEXEXTRACT(text(V4:V, "0.00"), "[0-9.]+")),
lower(REGEXEXTRACT(V4:V, "[A-Za-z]+")) = "cm", VALUE(REGEXEXTRACT(text(V4:V, "0.00"), "[0-9.]+"))/100,
lower(REGEXEXTRACT(V4:V, "[A-Za-z]+")) = "mm", VALUE(REGEXEXTRACT(text(V4:V, "0.00"), "[0-9.]+"))/1000)),
counta(A4:A)+1, 1)

This checks if Metres is blank, if so it takes the number values from the feet column (I have a regex to extract numbers here because staff sometimes write 3ft 2in) and adds it to the inches/12 to create a decimal feet and inches, it's then *0.3048 to convert to metres.

If the feet are blank then it moves on to Metric. Staff can enter metric as m, cm, or mm so I have to regex out the text and check if it's metres, if so return the regex numbers, if not, check for cm and then /100 to convert to metres. Same again for mm. This means staff can enter data how they're used to and the formula will convert any of the inputs to metres.

Cell AF4 is similar converting height into metres, but with another twist:
=array_constrain(ArrayFormula(
IF(isblank(X4:X), 
if(not(isblank(W4:W)), 
CONVERT(VALUE(REGEXEXTRACT(text(W4:W, "0.00"), "[0-9.]+")), lower(REGEXEXTRACT(W4:W, "[A-Za-z]+")), "m"),

if(isblank(AA4:AA), "",
if(isblank(AC4:AC), 
((VALUE(REGEXEXTRACT(text(AA4:AA, "0.00"), "[0-9.]+"))+(VALUE(REGEXEXTRACT(text(AB4:AB, "0.00"), "[0-9.]+"))/12))*0.3048),

if(
((VALUE(REGEXEXTRACT(text(AA4:AA, "0.00"), "[0-9.]+"))+(VALUE(REGEXEXTRACT(text(AB4:AB, "0.00"), "[0-9.]+"))/12))*0.3048)
>= ((VALUE(REGEXEXTRACT(text(AC4:AC, "0.00"), "[0-9.]+"))+(VALUE(REGEXEXTRACT(text(AD4:AD, "0.00"), "[0-9.]+"))/12))*0.3048),
((VALUE(REGEXEXTRACT(text(AA4:AA, "0.00"), "[0-9.]+"))+(VALUE(REGEXEXTRACT(text(AB4:AB, "0.00"), "[0-9.]+"))/12))*0.3048), 
((VALUE(REGEXEXTRACT(text(AC4:AC, "0.00"), "[0-9.]+"))+(VALUE(REGEXEXTRACT(text(AD4:AD, "0.00"), "[0-9.]+"))/12))*0.3048))

))), 
if(CONVERT(VALUE(REGEXEXTRACT(text(W4:W, "0.00"), "[0-9.]+")), lower(REGEXEXTRACT(W4:W, "[A-Za-z]+")), "m") 
>= CONVERT(VALUE(REGEXEXTRACT(text(X4:X, "0.00"), "[0-9.]+")), lower(REGEXEXTRACT(X4:X, "[A-Za-z]+")), "m"), 

CONVERT(VALUE(REGEXEXTRACT(text(W4:W, "0.00"), "[0-9.]+")), lower(REGEXEXTRACT(W4:W, "[A-Za-z]+")), "m"), 

CONVERT(VALUE(REGEXEXTRACT(text(X4:X, "0.00"), "[0-9.]+")), lower(REGEXEXTRACT(X4:X, "[A-Za-z]+")), "m")

))), counta(A4:A)+1, 1)

So for height the added complexity is that there is a left and right height, either one of which could be the longest. We only want the longest. This means I had to do the same formulas but nest them within an IF statement that asks which result is greater and then returns the correct result. This has to be done for meters and feet.

Because it's sheets you can't assign variables. In a language such as JavaScript I'd be able to work out feet or inches as a function, then convert to metres as a function and assign that value to a variable for both heights, compare both variables and return the heighest. However, because it's sheets the only solution is to repeat the code over and over - or split the calculation across multiple rows, with each step being a row. This is a bit easier to read, but much more error prone in sheets (especially if the end user isn't the person who coded the sheet, as is the case here).

I created and managed countless google sheet solutions of compariable complexity throughout my career, part of the reason I'm learning to code is because I enjoyed it so much.
