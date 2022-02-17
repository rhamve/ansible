# Use case
## Sending Same emails to everyone by CEO
## Examples
### HTML with template / variables
### Ansible with template / variables
Example mysql configuration template file with variables will outcome a valid configuration file

## Jinja2
Jinja2 template is a modern and designer-friendly templating language for Python.

## String manipulation - FILTERS
The name is {{ my_name }} => The name is Bond
The name is {{ my_name | uppper }} => The name is BOND
The name is {{ my_name | lower }} => The name is bond
The name is {{ my_name | title }} => The name is Bond
The name is {{ my_name | replace ("Bond","Bourne") }} => The name is Bourne
The name is {{ first_name | default("James") }} {{ my_name }} => The name is James Bond

## Filter - List and Set
min, max, unique, union, intersect, random, join

## Loops
{% for number in [0,1,2,3,4] %}
{{ number }}
{% endfor %}

## Conditions
{% for number in [0,1,2,3,4] %}
    {% if number == 2 %}
        {{ number }}
    {% endif %}
{% endfor %}
