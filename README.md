<a href="https://www.packtpub.com/en-us/unlock"><img src="https://drive.google.com/uc?export=view&id=1lQCTQQ8iV5pGuPA1n5wuds-3pwJi0OD_"></a>
<h1 align="center">
Python Object-Oriented Programming, Fifth Edition</h1>
<p align="center">This is the code repository for <a href ="https://www.packtpub.com/en-us/product/python-object-oriented-programming-fifth-edition/9781836642596"> Python Object-Oriented Programming, Fifth Edition</a>, published by Packt.
</p>

<h2 align="center">
Learn how and when to apply OOP principles to build scalable and maintainable Python applications
</h2>
<p align="center">
Steven F. Lott, Dusty Phillips</p>

<p align="center">
   <a href="https://packt.link/dHrHU" alt="Discord" title="Learn more on the Discord server"><img width="32px" src="https://cliply.co/wp-content/uploads/2021/08/372108630_DISCORD_LOGO_400.gif"/></a>
  &#8287;&#8287;&#8287;&#8287;&#8287;
  <a href="https://packt.link/free-ebook/9781836642596"><img width="32px" alt="Free PDF" title="Free PDF" src="https://cdn-icons-png.flaticon.com/512/4726/4726010.png"/></a>
 &#8287;&#8287;&#8287;&#8287;&#8287;
  <a href="https://packt.link/gbp/9781836642596"><img width="32px" alt="Graphic Bundle" title="Graphic Bundle" src="https://cdn-icons-png.flaticon.com/512/2659/2659360.png"/></a>
  &#8287;&#8287;&#8287;&#8287;&#8287;
   <a href="https://www.amazon.in/Python-Object-Oriented-Programming-maintainable-applications/dp/1836642598/ref=sr_1_2?crid=2S88CJUM2CJJU&dib=eyJ2IjoiMSJ9.sygyVdoBdW8dLm-3FqzkpgBvrPEHj1DeUYhNQtyW6VhWTIB-gm69wLgL_yBXIkm9uO4qoGpk0EsuvBSynPcisIsMlrUb_TjSukke-Kxq2N5Wa9aVcLb8uB1MNTH3H58YNhYU3diW7g1XSNC-OQXUBGzh1qFi32gv_cLsIuuZ7Behs9PN4A5iOzOKwBq0SPUENsRh42jLVNll7gYHbdTcR_a45nlxIe_M93eULiLxShw.pgI9C2r9sydE5A7qjkKpATOxQ2N2XGLmhkqC-t1Rh1s&dib_tag=se&keywords=Python+Object-Oriented+Programming&qid=1768981409&sprefix=python+object-oriented+programming%2Caps%2C385&sr=8-2"><img width="32px" alt="Amazon" title="Get your copy" src="https://cdn-icons-png.flaticon.com/512/15466/15466027.png"/></a>
  &#8287;&#8287;&#8287;&#8287;&#8287;
</p>
<details open> 
  <summary><h2>About the book</summary>
<a href="https://www.packtpub.com/en-us/product/python-object-oriented-programming-fifth-edition/9781836642596">
<img src="https://content.packt.com/B31856/cover_image_small.jpg" alt="Python Object-Oriented Programming, Fifth Edition" height="256px" align="right">
</a>

Learn to write effective, maintainable, and scalable Python applications by mastering object-oriented programming with this updated fifth edition. Whether you’re transitioning from scripting to structured development or refining your OOP skills, this book offers a clear, practical path forward.
You’ll explore Python’s approach to OOP, from class creation and inheritance to polymorphism and abstraction, while discovering how to make smarter decisions about when and how to use these tools. You’ll apply what you learn through hands-on examples and exercises.
Updated for Python 3.13, this edition simplifies complex topics such as abstract base classes, testing with unittest and pytest, and async programming with asyncio. It introduces a new chapter on Python’s type hinting ecosystem—crucial for modern Python development.
Written by long-time Python experts Steven Lott and Dusty Phillips, this edition emphasizes clarity, testability, and professional software engineering practices. It helps you move beyond scripting to building well-structured, production-ready Python systems.
By the end of this book, you’ll be confident in applying OOP principles, design patterns, type hints, and concurrency tools to create robust and maintainable Python applications.</details>
<details open> 
  <summary><h2>Key Learnings</summary>
<ul>

<li>Write Python classes and implement object behaviors</li>

<li>Apply inheritance, polymorphism, and composition</li>

<li>Understand when to use OOP—and when not to</li>

<li>Use type hints and perform static and runtime checks</li>

<li>Explore common and advanced design patterns in Python</li>

<li>Write unit and integration tests with unittest and pytest</li>

<li>Implement concurrency with asyncio, futures, and threads</li>

<li>Refactor procedural code into well-designed OOP structures</li>

</ul>

  </details>

<details open> 
  <summary><h2>Chapters</summary>
     <img src="https://cliply.co/wp-content/uploads/2020/02/372002150_DOCUMENTS_400px.gif" alt="Unity Cookbook, Fifth Edition" height="556px" align="right">
<ol>

  <li>Object-Oriented Design</li>

  <li>Objects in Python</li>

  <li>When Objects Are Alike</li>

  <li>Expecting the Unexpected</li>

  <li>When to Use Object-Oriented Programming</li>

  <li>Abstract Base Classes and Operator Overloading</li>

  <li>Python Type Hints</li>

  <li>Python Data Structures</li>

  <li>The Intersection of Object-Oriented and Functional Programming</li>

  <li>The Iterator Pattern</li>

  <li>Common Design Patterns</li>

  <li>Advanced Design Patterns</li>

  <li>Testing Object-Oriented Programs</li>

  <li>Concurrency</li>

</ol>

</details>


<details open> 
  <summary><h2>Requirements for this book</summary>
<ul>
<li>All the examples were tested with Python 3.12.5. The pyright tool, version 1.1, was used to confirm
that the type hints were consistent.</li>
<li>Some of the examples depend on an internet connection to gather data. These interactions with
websites generally involve small downloads.</li>
<li>Some of the examples involve packages that are not part of Python’s built-in standard library. In
the relevant chapters, we note the packages and provide the install instructions. All of these extra
packages are in the Python Package Index, at https://pypi.org.</li>
  </ul>
  </details>

<details open> 
  <summary><h2>Errata & Troubleshooting Tips</summary>

### `Page 440` (**Imitating objects using mocks**): 

The Status class defines an enumeration of `three` string values. We’ve provided symbolic names
such as Status.CANCELLED so that we can have a finite, bounded domain of valid status values. The
actual values stored in the database will be strings such as “CANCELLED” that — for now — happen
to match the symbols we’ll be using in the application. In the future, the domain of values may
expand or change, but we’d like to keep our application’s symbolic names separate from the strings
that appear in the database. It’s common to use numeric codes with Enum, but they can be difficult
to remember.

<details> 
  <summary><h2>Get to know Authors</h2></summary>

_Steven F. Lott_ Steven Lott has been programming since computers were large, expensive, and rare. Working for decades in high tech has given him exposure to a lot of ideas and techniques, some bad, but most are helpful to others. Since the 1990s, Steven has been engaged with Python, crafting an array of indispensable tools and applications. His profound expertise has led him to contribute significantly to Packt Publishing, penning notable titles like "Mastering Object-Oriented," "The Modern Python Cookbook," and "Functional Python Programming." A self-proclaimed technomad, Steven's unconventional lifestyle sees him travelling back and forth across the US. He tries to live by the words “Don't come home until you have a story.”

_Dusty Phillips_ Dusty Phillips is a Canadian software developer known for authoring several popular programming books.



</details>
<details> 
  <summary><h2>Other Related Books</h2></summary>
<ul>

  <li><a href="https://www.packtpub.com/en-us/product/mastering-python-2e-second-edition/9781800207721">Mastering Python 2E, Second Edition</a></li>

  <li><a href="https://www.packtpub.com/en-us/product/python-object-oriented-programming-fourth-edition/9781801077262">Python Object-Oriented Programming, Fourth Edition</a></li>
 
</ul>

</details>

<details>

<summary><h2>Testing the code base</summary></h2>

This was tested using **uv**.

See https://docs.astral.sh/uv/ for how to install **uv**.

Each chapter is a separate mini-project.
Most a scripts, a few are libraries.

Generally, it's possible to use terminal commands like the following to confirm the chapter's code works:

```bash
cd ch_1
uvx tox run
```

This will install a copy of ``tox`` and run it to confirm the various virtual environments work.

</details>

<details>
<summary><h2>The project structure</summary></h2>

Each chapter's code is in a separate directory, `ch_01`, `ch_02`, etc.

Within the chapter, there's some combination of `src`, and `tests` folders.
There will also be a `pyproject.toml` file with parameters used to control tools
like **tox**.
</details>
<details>
<summary><h2>Download a free PDF</summary></h2>

 <i>If you have already purchased a print or Kindle version of this book, you can get a DRM-free PDF version at no cost.<br>Simply click on the link to claim your free PDF.</i>
<p align="center"> <a href="https://packt.link/free-ebook/9781836642596">https://packt.link/free-ebook/9781836642596</a> </p>

</details>
