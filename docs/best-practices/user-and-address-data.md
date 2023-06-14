## Name

**Use one combined field for first/last/middle/whatever name.**

:   Rationale (as given by the [HTML spec](https://www.w3.org/TR/html52/sec-forms.html#autofill-field) relating to
autofill):

    > For example, while it is common in some Western cultures to have a given name and a family name, in that order (and thus often referred to as a first name and a surname), many cultures put the family name first and the given name second, and many others simply have one name (a mononym). Having a single field is therefore more flexible.

    More enlightening facts on what names (don't) look like can be found in [Falsehoods Programmers Believe About Names](http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/).

## Email address

**Do _not_ use the email address as the primary key of a database table.**

:   People may change their email address.

**Store in lowercase and query in lowercase.**

:   You don't want someone mistakenly signing up _again_ with a slight lowercase/uppercase variation of his email
address.

    The domain part of the address is case-insensitive anyway (because that's the way the [domain name system](https://en.wikipedia.org/wiki/Domain_Name_System) works).

    The local part (left of the @ character) is handled case-insensitively by most systems anyway.

**Do not try to "verify" the formal correctness of an email address.**

:   The relevant RFC standards allow so many weird combinations that it's nearly impossible to do that correctly. But
even if you did, that work is pointless in the end: even a formally correct address might just not exist.

**However, you _may_ want to add a basic format check.**

:   A simple check like "has exactly one @ character with at least one preceding
character and three characters following it" can help prevent people from mixing up the form fields. Also, it should
catch the most obvious typing errors.

**Do _not_ try to look up whether the domain name of an email address is valid.**

:   Even if it is, it might not accept mail.

    You will not do a better job in this than the developers of [mail transfer agent](https://en.wikipedia.org/wiki/Message_transfer_agent) software. Most likely, you will mess it up for complicated things like [IDN domains](https://en.wikipedia.org/wiki/Internationalized_domain_name).

**Do not even try to verify whether the given top level domain (e.g. `.de`) is valid.**

:   The list of [top level domains](https://en.wikipedia.org/wiki/Top-level_domain) is extended all the time. Do you
want to lock out early adopters?

**Always verify ownership.**

:   Whenever the user supplies an email address, that value can only be used after the user clicked a confirmation link
sent there. This applies both to initial signup and to the user changing their email address later.

## Password

**Do not impose a length limit.**

:   The "traditional" kind of length limits like 16, 20 or 30 does not have any advantages. As passwords are stored as a
hash anyway, a 100-character password and one that has 10 characters take up the same amount of space in the database.
However, those limits can prevent people from using certain strong passwords.

    That said, you may want to put in a safety net against attackers uploading "passwords" consisting of several megabytes. However, be sure to set the limit well over 100 characters.

**Do not force people to use certain character classes.**

:   Rules like "has to have lowercase and uppercase letters, digits and a special character" make it harder to memorize
passwords, restrict the amount of randomness contained in the password (easing brute force attacks) and get in the way
of some (secure) password schemes.

**Do not disallow any kinds of characters.**

:   Even "special" characters like spaces, quotes and angle brackets will not cause problems because the password is
hashed anyway.

**Enforce a minimum password length, say 10 characters.**

:   While it's not impossible to create shorter passwords with sufficient randomness, you should not assume that your
users are capable of doing so.

    For administrator accounts, your application could use a separate higher minimum, e.g. 16 or 20 characters. 

**Consider rejecting newly set passwords that are public knowledge.**

:   Hundreds of millions of real-world passwords from leaks like database dumps have been collected. Malicious actors
can and will try those first, significantly speeding up their brute-force attacks.

    On the other hand, when the user chooses a password, instead of accepting it blindly and hoping for the best, we can easily check whether that password is on those lists. In fact, that is even recommended by a US government standard from 2017! [NIST Special Publication 800-63B](https://pages.nist.gov/800-63-3/sp800-63b.html#-5112-memorized-secret-verifiers) says:

    > When processing requests to establish and change memorized secrets, verifiers SHALL compare the prospective secrets against a list that contains values known to be commonly-used, expected, or compromised. For example, the list MAY include, but is not limited to:

    > • Passwords obtained from previous breach corpuses.

    > (…)

    > If the chosen secret is found in the list, the CSP or verifier SHALL advise the subscriber that they need to select a different secret, SHALL provide the reason for rejection, and SHALL require the subscriber to choose a different value.

**When hashing or verifying the password, convert the characters to bytes via UTF-8.**

:   This will make sure passwords with special characters like umlauts will work.

## Username / Login name

**If possible, avoid having a separate username field and use the e-mail address instead.**

:   People forget their usernames all the time, but usually remember their email address. Also, most systems will ask
for the email address anyway, so there's not much reason to require an additional piece of uniquely identifying
information.

However, if you _do_ need usernames...

**Store username in lowercase and query it in lowercase.**

:   You don't want someone (else) signing up with a slight lowercase/uppercase variation of an existing username.

**Do not use the username as primary key.**

:   People may want to change their username, in which case you don't want to update foreign keys in other tables.

## Zip code

**Store the zip code as a string.**

:   Some people think of zip codes as numbers. This won't even work for Germany, where millions of people live in areas
with a zip code starting with a zero.

**Do _not_ make the zip code field mandatory.**

:   Not all countries even have zip code systems.

**Do _not_ verify the formal correctness of a zip code unless you really, really have to.**

:   It's mostly pointless - even a formally correct zip code might not exist, or still be a typo.

## Street address

**Use one combined field for the street address, or one field per street address line.**

:   Leaving things like post-office boxes aside, not every street in the world even _has_ numbers. Even if it did, what
do you want to do with the separate info? It's not like you need to sort the numbers in a street to optimize the
postman's route, right?

## Phone number

**Beyond the country code, do not assume any kind of structure - use one combined text input field.**

:   For example, there is no international standard regarding area or city code.

**Let people use whatever format they are used to.**

:   Commonly used conventions vary greatly around the world and may include several non-digit characters you probably do
not think of.

**Store the phone number as a string, exactly as entered, and do not try to normalize its format.**

:   If you pass the number to another system, you could still apply required normalization steps on the fly.
