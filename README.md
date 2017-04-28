# dental
This is a small project to testify the benefits claimed from the dental insurance package and find the optimal package for the specific need.

First, the terminology in the dental insurance package is determined (based upon my personal understanding). 
* Service categories: different category has different insurance coverage which varies with individual companies and plans
  1. Preventative services: routine cleaning, X-ray exams, and the other necessary preventative items. The annual visit limit may apply.
  2. Basic restorative services: filling.
  3. Major restorative services: crown and bridging.
  4. Orthodontia: adolescent only, which also has a life maximum amount, usually at \$1000.
* Coverage
  1. Deductible: the amount of fee that is paid by the patient before the coverage of the insurance is invoked.
    - total deductible and per category deductible
  2. Annual maximum benefit: the total amount of fee that the insurance covers after the deductible and self-paid part.
* Acceptance: 
  1. In-network and out-network: many plans refuse to pay the clinic visits to out-network dentists. The network size and the average level of the dentists in the network may vary.
  2. Sometimes, the difficulty of the clamation of the insurance is another factor when choosing the plan. Some company may be more flexible than the others as mentioned by some clinic staff.

Next, I collected serveral dental insurance plans that are available in the market which include
  * Carefirst (2)
  * Metropolitan Life (3 individual + 1 group member as the benchmark)
  * Delta (dropped later as it is based on per visit copayment which is quite different from the other types, not sure about its acceptance by the dentists) 
  * Guardian (2)
- Above plan rates are based on the Internet search in April 2017 for a quota of an male adult at the age of 34 in Montgomery County, Maryland starting coverage from 06/01/2017. The plan details are also uploaded into this project folder.

The following are assumptions used in the comparison model:
1) The adult receives preventive services (exam and cleanings) twice a year at a constant rate, unitCostPrev (unit: \$/visit). The insurance package have a deductible for the preventive services at prevDeduct (unit: \$) and a copayment coverage of rPrev in [0, 1], typically 50%-100%.
2) The adult only needs some basic restorative services beyond preventive services, like fillings and simple surgery for extraction.
3) The cost of basic restorative services per year is set in a range from 0 to \$2000 per year.
4) There is a deductible for the basic services at baseDeduct (unit: \$) and the cost beyond the deductible will be paid by the insurance company at the coverage ratio of rBase in [0, 1].
5) The total paid coverage by the insurance company is bounded by the annual maximum benefit, maxY (unit: \$), which depends on the selection of the dental insurance package ranging from $500 to $2000.
6) In the cases when the deductible is shared by preventive services and basic restorative services, we assume that the preventive services will first claim the deductible. The coverage of basic services will be triggered immediately  when the preventive services use up the deductible, which is usually the case.

The code is as follows along with comments:

Only the matplotlib is used to output the comparison result
```python
import matplotlib.pyplot as plt
```

Base on the assumption, the function, nonPremCost, is developed to calculate the annual out-of-pocket dental expense for a single adult excluding the monthly premium.
```python
def nonPremCost(maxY, unitCostPrev, rPrev, prevDeduct, costBase, rBase, baseDeduct):
    paidPrev = 0
    paidBase = 0
    
    # first, calculate the cost paid by the patient for preventive services
    totalCostPrev = unitCostPrev * 2 # twice a year
    coveredPrev = min(maxY, max(0, totalCostPrev - prevDeduct) * rPrev) # the part covered by the insurance
    #print('coveredPrev=', coveredPrev)
    coveredBase = maxY - coveredPrev # the left coverable benefit for basic restorative services
    paidPrev = totalCostPrev - coveredPrev;
    #print('paidPrev=', paidPrev)
    
    # second, calculate the basic restorative services, suppose that total cost is costBase
    if coveredBase <= 0:
        paidBase = costBase
    else:
        paidBase = costBase - min(coveredBase, rBase*max(0, costBase-baseDeduct))
        
    extraCost = paidPrev + paidBase
    
    return extraCost  
```

The data from dental plans are used to make a comparison.
```python
unitCostPrev = 150 # cost estimate for each teeth cleaning
x = [50*i for i in range(21)] # vector for total base cost


# BlueCare lower $28.93
# nonPremCost(1000, unitCostPrev, 1, 100, costBase, 0.8, 100)
yB1 = [nonPremCost(1000, unitCostPrev, 1, 100, costBase, 0.8, 100) for costBase in x]
yB1t = [i+28.93*12 for i in yB1] # total out-of-pocket (absolute)
yB1a = [i/.7+28.93*12 for i in yB1] # adjusted to pre-tax cost

# BlueCare Higher $ 44.76
# nonPremCost(1000, unitCostPrev, 1, 0, costBase, 0.8, 60)
yB2 = [nonPremCost(1000, unitCostPrev, 1, 0, costBase, 0.8, 60) for costBase in x]
yB2t = [i+44.76*12 for i in yB2]
yB2a = [i/.7+44.76*12 for i in yB2]

# Metlife Option 1 $33.62
# nonPremCost(1000, unitCostPrev, 1, 0, costBase, 0.7, 75)
yM1 = [nonPremCost(1000, unitCostPrev, 1, 0, costBase, 0.7, 75) for costBase in x]
yM1t = [i+33.62*12 for i in yM1]
yM1a = [i/.7+33.62*12 for i in yM1]

# Metlife Option 2 $37.5
# nonPremCost(1500, unitCostPrev, 1, 0, costBase, 0.7, 50)
yM2 = [nonPremCost(1500, unitCostPrev, 1, 0, costBase, 0.7, 50) for costBase in x]
yM2t = [i+37.5*12 for i in yM2]
yM2a = [i/.7+37.5*12 for i in yM2]

# Metlife Option 3 $42.36
# nonPremCost(2000, unitCostPrev, 1, 0, costBase, 0.8, 25)
yM3 = [nonPremCost(2000, unitCostPrev, 1, 0, costBase, 0.8, 25) for costBase in x]
yM3t = [i+42.36*12 for i in yM3]
yM3a = [i/.7+42.36*12 for i in yM3]

# Guardian Option 1 $ 37.07
# nonPremCost(1000, unitCostPrev, 1, 50, costBase, 0.7, 0)
yG1 = [nonPremCost(1000, unitCostPrev, 1, 50, costBase, 0.7, 0) for costBase in x]
yG1t = [i+37.07*12 for i in yG1]
yG1a = [i/.7+37.07*12 for i in yG1]

# Guardian Option 2 $ 25.50
# nonPremCost(500, unitCostPrev, 0.8, 50, costBase, 0.5, 0)
yG2 = [nonPremCost(500, unitCostPrev, 0.8, 50, costBase, 0.5, 0) for costBase in x]
yG2t = [i+25.50*12 for i in yG2]
yG2a = [i/.7+25.50*12 for i in yG2]

# Metlife Option 4 $37.5
# This is a group dental plan which is unlike all above ones which are sold for individual members.
# nonPremCost(1500, unitCostPrev, 1, 0, costBase, 0.8, 50)
yMg = [nonPremCost(1500, unitCostPrev, 1, 0, costBase, 0.8, 50) for costBase in x]
yMgt = [i+37.5*12 for i in yMg]
yMga = [i/.7+37.5*12 for i in yMg]

y = [i+2*unitCostPrev for i in x]
```

The output can be selected from three options:
* comparison based on the patient self-paid fee per year excluding the monthly premium
* comparison based on the total cost of the patient including the pre-tax monthly premium
* comparison based on the total cost of the patient which has been adjusted to pre-tax cost given the personal income tax rate at 30%
```python
plotChoice = 1

if plotChoice == 0:
    plt.plot(x,yB1,'b--',x,yB2,'b^',x,yM1,'r--',x,yM2,'r^',x,yM3,'r*',x,yG1,'g--', x,yG2,'g^', x, yMg, 'ro')
    plt.title('Extra Dental Cost when preventive cost: %s' %(unitCostPrev*2))
elif plotChoice == 1:
    plt.plot(x,y,'k', x,yB1t,'b--',x,yB2t,'b^',x,yM1t,'r--',x,yM2t,'r^',x,yM3t,'r*',x,yG1t,'g--', x,yG2t,'g^')
    plt.title('Dental Cost in the bill when preventive cost: %s' %(unitCostPrev*2))
elif plotChoice == 2:
    plt.plot(x,y,'k',x,yB1a,'b--',x,yB2a,'b^',x,yM1a,'r--',x,yM2a,'r^',x,yM3a,'r*',x,yG1a,'g--', x,yG2a,'g^', x, yMga, 'ro')
    plt.title('Adjusted Pre-Tax Dental Cost when preventive cost: %s' %(unitCostPrev*2))
else:
    print('no figure generateds')

plt.show()
```
The complete code can be found in the Python Notebook file with the name of dental_plan_comp.

![alt text](https://github.com/ykLIU1982/dental/blob/master/figure_1.png "Total cost as shown in Option 1")


The results will show some plan which has a higher monthly premium but will save the patient more money at a higher coverage rate and higher annual maximum benefits if the patient estimates a higher dental need in the coming year. If the patient only has very limited dental care, such as routing cleaning for twice a year, maybe he or she can just exempt from the insurance by paying the totoal visit cost. Furthermore, the group insurance usually outperforms the individual ones thanks to the group negotiation advatages. 

