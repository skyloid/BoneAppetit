# BoneAppetit

// /src/services/firebaseApi.js
import firestore from '@react-native-firebase/firestore';

/**
 * Add a person to a family tree
 * @param {Object} personData - The person's data (firstName, lastName, biography, etc.)
 * @param {string} familyTreeId - The ID of the family tree to add the person to
 * @returns {string|null} - The ID of the newly created document or null on failure
 */
export const addPersonToTree = async (personData, familyTreeId) => {
  try {
    // Validate inputs
    if (!personData || !familyTreeId) {
      throw new Error('Person data and family tree ID are required');
    }

    // Add timestamp to the person data
    const personWithTimestamp = {
      ...personData,
      createdAt: firestore.FieldValue.serverTimestamp(),
      updatedAt: firestore.FieldValue.serverTimestamp(),
    };

    // Reference to the persons subcollection within the specific family tree
    const personsCollectionRef = firestore()
      .collection('familyTrees')
      .doc(familyTreeId)
      .collection('persons');

    // Add the person document
    const docRef = await personsCollectionRef.add(personWithTimestamp);

    console.log('Person successfully added with ID:', docRef.id);
    return docRef.id;
  } catch (error) {
    console.error('Error adding person to tree:', error);
    return null;
  }
};

/**
 * Get all persons in a family tree
 * @param {string} familyTreeId - The ID of the family tree
 * @returns {Array|null} - Array of persons or null on failure
 */
export const getPersonsFromTree = async (familyTreeId) => {
  try {
    if (!familyTreeId) {
      throw new Error('Family tree ID is required');
    }

    const personsSnapshot = await firestore()
      .collection('familyTrees')
      .doc(familyTreeId)
      .collection('persons')
      .orderBy('createdAt', 'desc')
      .get();

    const persons = [];
    personsSnapshot.forEach((doc) => {
      persons.push({
        id: doc.id,
        ...doc.data(),
      });
    });

    return persons;
  } catch (error) {
    console.error('Error fetching persons from tree:', error);
    return null;
  }
};

/**
 * Update a person's information
 * @param {string} familyTreeId - The ID of the family tree
 * @param {string} personId - The ID of the person to update
 * @param {Object} updateData - The data to update
 * @returns {boolean} - True on success, false on failure
 */
export const updatePerson = async (familyTreeId, personId, updateData) => {
  try {
    if (!familyTreeId || !personId || !updateData) {
      throw new Error('Family tree ID, person ID, and update data are required');
    }

    // Add updated timestamp
    const dataWithTimestamp = {
      ...updateData,
      updatedAt: firestore.FieldValue.serverTimestamp(),
    };

    await firestore()
      .collection('familyTrees')
      .doc(familyTreeId)
      .collection('persons')
      .doc(personId)
      .update(dataWithTimestamp);

    console.log('Person successfully updated');
    return true;
  } catch (error) {
    console.error('Error updating person:', error);
    return false;
  }
};

/**
 * Delete a person from a family tree
 * @param {string} familyTreeId - The ID of the family tree
 * @param {string} personId - The ID of the person to delete
 * @returns {boolean} - True on success, false on failure
 */
export const deletePerson = async (familyTreeId, personId) => {
  try {
    if (!familyTreeId || !personId) {
      throw new Error('Family tree ID and person ID are required');
    }

    await firestore()
      .collection('familyTrees')
      .doc(familyTreeId)
      .collection('persons')
      .doc(personId)
      .delete();

    console.log('Person successfully deleted');
    return true;
  } catch (error) {
    console.error('Error deleting person:', error);
    return false;
  }
};

/**
 * Create a new family tree
 * @param {Object} treeData - The family tree data (name, description, etc.)
 * @param {string} userId - The ID of the user creating the tree
 * @returns {string|null} - The ID of the newly created tree or null on failure
 */
export const createFamilyTree = async (treeData, userId) => {
  try {
    if (!treeData || !userId) {
      throw new Error('Tree data and user ID are required');
    }

    const treeWithMetadata = {
      ...treeData,
      ownerId: userId,
      memberIds: [userId],
      createdAt: firestore.FieldValue.serverTimestamp(),
      updatedAt: firestore.FieldValue.serverTimestamp(),
    };

    // Create the family tree document
    const treeRef = await firestore()
      .collection('familyTrees')
      .add(treeWithMetadata);

    // Update the user's document with the familyTreeId
    await firestore()
      .collection('users')
      .doc(userId)
      .update({
        familyTreeId: treeRef.id,
        updatedAt: firestore.FieldValue.serverTimestamp(),
      });

    console.log('Family tree successfully created with ID:', treeRef.id);
    return treeRef.id;
  } catch (error) {
    console.error('Error creating family tree:', error);
    return null;
  }
};
