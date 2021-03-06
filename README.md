# Association-Rule-Mining

TEAM 9
Ashwin Tamilselvan (at3103)
Niharika Purbey (np2544)

Document Structure:
```
main.py: The main driver program. It takes care of user input/interaction, vectorizing the dataset and calling the apriori algorithm to generate association rules.

example-run.txt: Output of an interesting sample run

algorithms - 
	apriori.py: The main algorithm [1] consisting of large Itemset generation, Candidate Itemset generation and High Confidence Association Rule Generation

Data_Set_gen -
	Data_prepare.py: Stratified Sampling of the original dataset
	pre_vectorzing.py: Converting attributes to different formats
	vectorizing.py: Vectorizes the dataset
	Integrated_data_set.csv: Integrated data set file
	vectorized_data_set.csv: Vectorized data set file

utils - 
	display.py: Prints the Frequent Itemset and the High Confidence Association Rules and also saves to an output file (output.txt)	
```

a) We used the 3-1-1 Call Center Inquiry dataset from the NYC Open Data set to generate the INTEGRATED-DATASET
file (https://data.cityofnewyork.us/view/tdd6-3ysr)

b) The original dataset contains 58,435,089 rows with information about call inquiries from 2010 - present day.
Since the dataset was huge, we decided to work on the data for Jan 2015 and Feb 2015 only. 

The original dataset had the columns: Unique Key, Date, Time, Agency, Inquiry Name, Brief Description, Call
Resolution. We deleted the column "Brief Description" because the descriptions were similar to "inquiry name",
hence redundant. 

To get interesting rules, we decided to focus on the top 5 agencies (i.e. top 5 agencies with maximum number of
tuples). The top 5 agencies were DOF, NYPD, 3-1-1, DSNY, HPD. 

This initial filtering was done on the website itself. 

After downloading this data, we had around 500,000 rows, so we decided to use a stratified sample of this data.
We performed stratified sampling based on the column "call resolution" to get a uniform distribution on the
values of "call resolution". We used the Python library 'sklearn' to do this. Script can be found in Data_Set
gen/Data_prepare.py     

To further generate interesting results, we converted the date to day of the week and converted the raw time to
designated time-zones i.e "Morning","Noon","Evening" and "Night" where "Morning":6am-12pm ; "Noon":12pm-6pm ;
"Evening":6pm-12am ; "Night":12am-6am. The script to perform this operation can be found in Data_Set_gen
pre_vectorizing.py     

c) 
We were interested in finding out what the call resolutions were for different types of inquiry. Also, we
wanted to know if there is a specific day of the week or a specific time when people made more inquiries or
people made more of a particular type of inquiry. 
It would also be interesting to know if certain call resolutions were linked to certain time of the day or day
of the week. Or if the agencies were linked to any particular call resolution or type of inquiry.

Since there is a lot of data on this on the NYC Dataset, it would be difficult to visualize interesting
patterns by simply viewing it in charts. Hence, we require association rules to find interesting relationships
between the different atrributes of the dataset. 

[DOF, Noon] => [Information Provided], Conf: 47.1957671958%, Supp: 3.969949441%
[Evening, DOF] => [Information Provided], Conf: 47.1355311355%, Supp: 5.72705262408%


d) To run :
```
python main.py <Integrated data set> <minimum support> <minimum confidence> [v] [r]
where 
	<Integrated data set> => Integrated_data_set.csv 
	minimum support => value between 0 and 1
	minimum confidence => value between 0 and 1
	[v] => (Optional) To show verbose results
	[r] => (Optional) To show restricted results {Rules with only Call Resolution on RHS}

Example run: python main.py Integrated_data_set.csv 0.01 0.04
```

e) Internal Design

1. For ease of use, we started by vectorizing the dataset. The script in Data_Set_gen/vectorizing.py performs
this operation. We saved the vectorized data as a CSV file, wherein each value in each of the columns were now
columns in this new CSV file.

2. We then ran the Apriori algorithm on the dataset. The Apriori algorithm implementation is as given in the
paper "Fast algorithms for Mining Association Rules" sections 2.1 and 2.1.1.

3. We first generate L1 i.e. 1-large itemsets by simply summing values of each of the columns in the vectorized
CSV file and checking whether it is greater than the minimum support. The columns in the vectorized CSV file
represent all the items. 

4. We then generate the candidate itemsets(C_k) by joining L_k-1 with itself. We then prune this set by
checking if the (k-1)—item subsets of this set are present in large (k-1)—itemset. Hence, after the prune
method the final C_k is generated.

5. We then calculate the support for each of the items in the set C_k by simply 'anding' the corresponding
columns from the vectorized dataset and summing the final result. From this, we retain all those items that
have support greater than minimum support in the Large itemset (L_k).   

6. We continue this process(steps 4 and 5) until the Large itemset generated(L_k) is null.

7. We then generate rules from the Large Itemsets i.e L1, L2, L3,... (till whenever the algorithm ran to). Here
we calculate the confidence of each rule as conf(LHS=>RHS) = sup(LHS U RHS)/sup(LHS). We retain only those
rules whose confidence is greater than the input minimum confidence. 

8. We then print these rules in decreasing order of confidence along with the frequent itemsets in decreasing
order of support. We also save this to an output file (output.txt). If the code is run with the verbose flag, a verbose output is generated in verbose_ouput.txt.


f) Interesting Results
```
python main.py Integrated_data_set.csv 0.01 0.3 

[Monday] => [Evening], Conf: 44.8522118627%, Supp: 8.05027415794%
[Friday] => [Evening], Conf: 44.7794779478%, Supp: 7.08538061668%
[Wednesday] => [Evening], Conf: 44.5170840015%, Supp: 6.42490920743%
[Thursday] => [Evening], Conf: 43.4210526316%, Supp: 7.1085238197%
[Tuesday] => [Evening], Conf: 43.1402273958%, Supp: 7.09250160222%
[Saturday] => [Evening], Conf: 39.0065604499%, Supp: 3.70469272947%
[Sunday] => [Evening], Conf: 38.8628260462%, Supp: 3.68689026561%


From the above results, we can infer that the majority of the calls placed were during the evening period(6 PM
- 12 AM). As the time zones form a mutually exclusive and exhaustive set, we can derive a clean breakup of the
various time zones of the day. 

The support of the above rules conforms to the hypothesis that the number of queries would be more during the
weekdays, as not all the agencies would be working over the weekend. Only urgent calls/queries are made
during the weekend, and the support values of Saturday/Sunday in comparision to the weekends are a good
indicative of it.

```

```
python main.py Integrated_data_set.csv 0.01 0.3 

[Noise from Neighbor, NYPD] => [CSMS SR], Conf: 93.4782608696%, Supp: 2.21996724347%
[Vehicle Blocking Driveway Complaint, NYPD] => [CSMS SR], Conf: 92.8070175439%, Supp: 1.88350067649%
[Morning, NYPD] => [CSMS SR], Conf: 55.7964970809%, Supp: 1.1909848323%
[Night, NYPD] => [CSMS SR], Conf: 41.4321339508%, Supp: 2.07042654703%
[Saturday, NYPD] => [CSMS SR], Conf: 41.0977242303%, Supp: 1.09307128107%

[NYPD] => [CSMS SR], Conf: 36.968766001%, Supp: 6.42668945382%

From the above results, we can infer that CSMS_SR(Call was resolved by submission of a Service Request) is the
most preferred resolutiom type for NYPD call inquiries. Between the third and fourth results, we can see that
the number of complaints/calls were more than double in the night in comparision to the ones in morning. 

And the last result provides us the requried evidence/aid to obtain the absolute number of calls for NYPD, that
were resolved via CSMR_SR. (≈6.4% of 50,000)

```

```
python main.py Integrated_data_set.csv 0.01 0.3 

[Heat or Hot Water Complaint in Entire Residential Building] => [HPD], Conf: 100.0%, Supp: 6.24888730639%
[Heat or Hot Water Complaint in Apartment] => [HPD], Conf: 100.0%, Supp: 5.10948905109%

[HPD], 19.9928787609%

The combined support for both the above rules is around 11% and the overall support for the frequent Itemset "HPD" (Department of Housing Preservation and Development) is 19% (i.e. 19% of 50,000). Hence we can see that out of the 19% inquiries for HPD, 11% were heat and hot water related. 
Looking at the above 2 rules, and the support for "HPD", we can conclude that most of the inquiries made for this Agency were heat and hot water related. 

```

REFERENCES:

[1] Agrawal R, Srikant R,Fast algorithms for mining association rules, 1994, VLDB (Vol. 1215, pp. 487-499).




