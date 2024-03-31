# problem_database

Contest problems database on [von](https://github.com/vEnhance/von) format (all problems in Portuguese). *Base de dados de problemas no formato do [von](https://github.com/vEnhance/von) (todos os problemas em português).*

In order to the PUIDs be generated correctly the function `inferPUID` in the `puid.py` file from von must be modified as shown below. *Para que os PUIDs sejam gerados corretamente a função `inferPUID` no arquivo `puid.py` do von deve ser modificada conforme mostrado abaixo.*

```py
# Regular expressions for Brazilian Olympiads
re_obmep = re.compile(r"OBMEP N(?P<level>[123]) F(?P<phase>[12]) (?P<year>\d{4})/(?P<problem>\d+)")
re_canguru = re.compile(r"Canguru (?P<category>Benjamim|Cadete|Júnior) (?P<year>\d{4})/(?P<problem>\d+)")
re_obm = re.compile(r"OBM (?P<year>\d{4})/(?P<problem>\d+)")
re_obm_banco = re.compile(r"OBM Banco (?P<year>\d{4})/(?P<problem>\d+)")

def inferPUID(source: str) -> str:
    source = source.replace("Finals", "F")
    # Check for the Brazilian Olympiad formats first
    for regex in [re_obmep, re_canguru, re_obm, re_obm_banco]:
        if (m := regex.match(source)) is not None:
            d = m.groupdict()
            # Extract the last two digits of the year
            year_last_two = d['year'][-2:]
            # Construct PUID based on specific format
            if regex == re_obmep:
                contest = "OBMEP"
                level_phase = f"N{d['level']}F{d['phase']}"
            elif regex == re_canguru:
                contest = f"Canguru{d['category']}"
                level_phase = ""
            elif regex == re_obm:
                contest = "OBM"
                level_phase = ""
            elif regex == re_obm_banco:
                contest = "OBMBanco"
                level_phase = ""
            source = f"{year_last_two}{lookup.get(contest, getOnlyAlphanum(contest))}{level_phase}P{getOnlyAlphanum(d['problem'])}"
            thresh = 11
            break
    else:
        # Your existing logic for other formats
        if source[0] == "H" and source[1:].isdigit():
            return source
        else:
            for k in sorted_lookup_keys:
                if source.startswith(k):
                    source = lookup[k] + source[len(k):]
                    thresh = 30
                    break
            else:
                thresh = 8
    source = getOnlyAlphanum(source)
    if len(source) <= thresh:
        return source
    else:
        # still too long, return some sort of hash
        return "Z" + (hashlib.sha256(source.encode("ascii")).hexdigest())[0:7].upper()
```
