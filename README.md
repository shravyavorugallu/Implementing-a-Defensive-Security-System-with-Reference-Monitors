# Reference_Monitor

```markdown
# Building a Security Layer

## Overview

This assignment will guide you through the process of creating a reference monitor using the security layer functionality in Repy V2. A reference monitor is an access control concept that mediates all access to objects by subjects. It can be used to allow, deny, or modify the behavior of calls made by a program.

In this assignment, an attacker attempts to write "MZ" to the first two characters of a file. Your goal is to prevent this attack by ensuring that "MZ" cannot be written to the start of any file. You'll do this by adding security rules to the functions available for reading and writing files.

Three design paradigms are at work in this assignment:

- **Accuracy**: The security layer should block only specific actions, such as writing "MZ" to the beginning of a file, while allowing other actions (e.g., writing "MX", "ZM") to proceed without interruption.
- **Efficiency**: The security layer should be resource-efficient to avoid performance degradation.
- **Security**: The security layer should be robust enough that an attacker cannot bypass it.

## Getting Started

### Prerequisites

- **Python**: You need Python 2.6 or 2.7. Instructions for installing Python on Windows can be found [here](https://www.python.org/downloads/release/python-2718/). If you're using Linux or macOS, Python should already be installed. Verify the version by running `python --version` in your terminal.
- **Repy V2**: You will need to install Repy V2, which is the runtime for building the security layer. Follow the instructions below to install Repy V2 from source.

### Installing Repy V2

1. Create a directory for the required Git repositories:
    ```bash
    mkdir SeattleTestbed
    cd SeattleTestbed
    ```

2. Clone the Repy V2 repository:
    ```bash
    git clone https://github.com/SeattleTestbed/repy_v2.git
    ```

3. Build Repy V2:
    ```bash
    cd repy_v2/scripts
    mkdir ~/path/to/build/dir
    python initialize.py
    python build.py ~/path/to/build/dir
    ```

4. Your build directory will contain the Repy V2 runtime, including `repy.py`, `restrictions.default`, and `encasementlib.r2py`.

### Running Repy Files

To run Repy files, use the following command:
```bash
python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [program].r2py
```

Make sure the Repy files end with `.r2py`.

## Building the Security Layer

### Creating a Basic Security Layer

A basic security layer intercepts file operations and checks whether the first two characters written to a file are "MZ". If they are, the security layer raises an exception to prevent the write operation.

```python
class SecureFile():
    def __init__(self, privilegedfo):
        self.privilegedfo = privilegedfo

    def readat(self, bytes, offset):
        return self.privilegedfo.readat(bytes, offset)

    def writeat(self, data, offset):
        if data.startswith("MZ") and offset == 0:
            raise ValueError("Cannot start file with MZ!")
        return self.privilegedfo.writeat(data, offset)

    def close(self):
        return self.privilegedfo.close()

def secure_openfile(filename, create):
    privilegedfo = openfile(filename, create)
    return SecureFile(privilegedfo)
```

### Testing the Security Layer

You will test your security layer by simulating an attacker trying to write "MZ" to a file. The attack should raise a `ValueError` if the security layer is working correctly.

```python
# Open a file
myfile = openfile("look.txt", True)

# Attempt to write "MZ" to the file
try:
    myfile.writeat("MZ", 0)
except ValueError:
    log("The security layer correctly blocked the write!!!")
else:
    log("Wrote an MZ!!!")

finally:
    # Close the file after our attempt.
    myfile.close()
```

### Code Analysis

The `writeat()` function in the `SecureFile` class checks if the data starts with "MZ" and if the offset is 0 (indicating the start of the file). If both conditions are met, it raises an exception to block the write. This approach follows the design principles of accuracy (only blocking "MZ"), efficiency (minimal resource usage), and security (blocking the attack).

### Running the Security Layer

To run your security layer with an attack program, use the following command:
```bash
python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [attack_program].r2py
```

## Extra Credit

For extra credit, implement a second security layer that prevents an attacker from writing the string "p0wnd" anywhere in a file. This will require handling the case where the string appears at any offset, not just at the beginning of the file.

### Submitting Your Work

- Turn in the Repy file for your reference monitor: `reference_monitor_[name_or_id].r2py`.
- For extra credit, submit the second Repy file: `extra_credit_[name_or_id].r2py`.

## Notes and Resources

- [Python Tutorial](https://docs.python.org/tutorial/)
- [Repy V2 API](https://seattle.poly.edu/wiki/RepyV2API)
- [Security Layer Tutorial](https://seattle.poly.edu/wiki/RepyV2SecurityLayers)
- [Thread Safety](http://en.wikipedia.org/wiki/Thread_safety)

```

This `README.md` provides a detailed guide for setting up and building the security layer for this assignment, as well as instructions for testing and submitting the work.
