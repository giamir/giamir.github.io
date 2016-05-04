---
title: A quirky name
layout: page
class:
---

If you ended up in this page it can mean 2 things.

* You accidentally clicked a link that you were not interested at.
* You want to know more about my quirky name (less likely).

Thinking about the origin of my name I realised my parents used a very specific algorithm to create it.
So as a developer I decided to write a short method in Ruby to explain it.
I know it’s geeky but don’t worry I also explained it in English if you’re too lazy to browse in the code.

{% highlight ruby %}
def quirky_name(dad_name, mom_name, baby_sex)
  return dad_name[0...3] + mom_name[0...3].downcase if baby_sex == :boy
  return mom_name[0...3] + dad_name[0...3].downcase if baby_sex == :girl
end

# Test Cases
Test.assert_equals(quirky_name("Gianfranco", "Mirella", :boy), "Giamir")
Test.assert_equals(quirky_name("Gianfranco", "Mirella", :girl), "Mirgia")
{% endhighlight %}

My forename is basically my father’s name first 3 letters joined to my mother’s name first 3 three letters. So my dad Gianfranco gave me the “Gia” and my mom Mirella gave me the “mir”.

Everyone can try to do what my parents did in 1990 but lots of time I guarantee it comes up a unpronounceable name. My parents explained to me that the “specs” of their original way to give a name to a child were to start with the dad’s name in case the baby was a boy or with the mom's name otherwise.

Anyway my name is Giamir and after 25 years I’m still happy with it.
It’s unique and I don’t need any nickname or stagename or similars.

I ❤ it.
