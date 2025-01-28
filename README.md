### Lab Project: Implementing a Defensive Security System with Reference Monitors

#### Objective
The goal of this lab is to help you understand security mechanisms by implementing a reference monitor using RepyV2. The reference monitor will act as a security layer that mediates all access to objects by subjects. Specifically, you will design a system that ensures the integrity of files by keeping a backup and verifying the validity of file contents before writing changes.

#### Overview
In this assignment, a security layer is created that ensures files are written correctly by checking if they start with 'S' and end with 'E'. If a file is invalid (i.e., it does not meet these criteria), the security layer will prevent it from being saved and revert to the backup file. This is especially useful in scenarios like firmware upgrades where incorrect writes can cause system failures.

#### Requirements
1. **Reference Monitor Functionality**:
   - When a file is opened using `ABopenfile()`, the reference monitor creates two copies of the file:
     - `filename.a` (the backup file)
     - `filename.b` (the file to be written to)
   - The reference monitor should verify that the file starts with 'S' and ends with 'E' when `close()` is called.
   - If the file is valid, the contents of `filename.b` will replace the original file.
   - If the file is invalid, the original file will retain the contents of `filename.a`.
   - Both `filename.a` and `filename.b` should be deleted after the operation, leaving only the original file.

2. **Security Constraints**:
   - The security layer should not modify or disable any of the RepyV2 API calls.
   - It should only prevent certain actions (invalid writes), while allowing other operations to proceed as normal.
   - The security layer should be efficient, using minimal resources and not storing unnecessary data in memory.

3. **File Validity**:
   - Files are considered valid if they start with 'S' and end with 'E'.
   - Any other characters (including lowercase 's', 'e', etc.) in the first or last position will make the file invalid.

4. **Test Cases**:
   - You will create test cases to ensure the reference monitor behaves correctly under various conditions and attacks.
   - The security layer should pass these tests without errors or unexpected outputs.

#### Steps to Implement

1. **Setup RepyV2**:
   - Build RepyV2 according to the SeattleTestbed Build Instructions.
   - Ensure you have Python 2.7 and RepyV2 installed correctly.

2. **Create the Security Layer**:
   - Implement the `ABFile` class to handle file operations, including creating backup files, writing to the file, and reading from the backup file.
   - Implement the `ABopenfile()` function to return an instance of `ABFile`.
   - Ensure that `close()` checks the validity of the file contents before replacing the original file.

3. **Test the Reference Monitor**:
   - Create test cases to verify that the security layer correctly handles valid and invalid file writes.
   - Test edge cases where the file contents are almost valid (e.g., 's' at the beginning or 'e' at the end).

4. **Penetration Testing**:
   - As an attacker, attempt to bypass the security layer by writing invalid data to the file or accessing the backup file directly.
   - Ensure that the security layer prevents such attacks.

#### Example Code for Reference Monitor

```python
class ABFile():
    def __init__(self, filename, create):
        self.Afn = filename + '.a'
        self.Bfn = filename + '.b'

        # Create files if needed
        if create:
            self.Afile = openfile(self.Afn, create)
            self.Bfile = openfile(self.Bfn, create)
            self.Afile.writeat('SE', 0)

    def writeat(self, data, offset):
        # Write to the B file
        self.Bfile.writeat(data, offset)

    def readat(self, bytes, offset):
        # Read from the A file (backup)
        return self.Afile.readat(bytes, offset)

    def close(self):
        # Validate the file and replace the original if valid
        if self.is_valid(self.Bfile):
            self.Afile.writeat(self.Bfile.readat(0, 0), 0)  # Replace original with B
        else:
            self.Afile.writeat(self.Afile.readat(0, 0), 0)  # Revert to backup
        self.Afile.close()
        self.Bfile.close()

    def is_valid(self, file):
        # Check if the file starts with 'S' and ends with 'E'
        content = file.readat(2, 0)
        return content[0] == 'S' and content[-1] == 'E'

def ABopenfile(filename, create):
    return ABFile(filename, create)
```

#### Example Test Case

```python
# Test Case 1: New File Operation

# Clean up existing files
if "testfile.txt.a" in listfiles():
    removefile("testfile.txt.a")
if "testfile.txt.b" in listfiles():
    removefile("testfile.txt.b")

# Open the file
myfile = ABopenfile("testfile.txt", True)  # Create an AB file

# Assert that the new file is valid ('SE')
assert 'SE' == myfile.readat(2, 0)

# Write valid data to the file
myfile.writeat("Stest12345E", 0)

# Assert that the file is still valid before closing
assert 'SE' == myfile.readat(2, 0)

# Close the file
myfile.close()
```

#### Running the Security Layer

To run the security layer and test it, use the following command:

```bash
python repy.py restrictions.default encasementlib.r2py [security_layer].r2py [attack_program].r2py
```

#### Deliverables

1. **Security Layer**: Submit the RepyV2 file `reference_monitor_[netid].r2py`.
2. **Test Cases**: Submit a zip file containing all the test cases you have created.
3. **Documentation**: Provide a brief explanation of your approach and any challenges you faced while implementing the reference monitor and testing it.
