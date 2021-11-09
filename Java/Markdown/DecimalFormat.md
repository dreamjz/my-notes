# DecimalFormat

**DecimalFormat is a concrete subclass of NumberFormat that formats decimal numbers.** It is designed to parse and format numbers in any locale. It also supports different kinds of numbers, including **integers(123)**, fixed-point **numbers (123.4)**, **scientific notation (1.23E4)**, **percentages (12%)**,  and **currency amounts ($123)**. 

To obtain a NumberFormat for a specific locale, including the default locale, call one of  NumberForamt's factory methods, do not call the DecimalFormat constructors directly, since the NumberFormat factory methods may return subclass other than DecimalFormat.

> If you want to specify how your numbers are formatted, then you must use the DecimalFormat constructor. If you want "the way most numbers are represented in my current locale", then use the default instance of NumberFormat.
>
> https://stackoverflow.com/questions/2296935/use-numberformat-or-decimalformat

