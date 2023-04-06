## React

Like useState but with delay time if value is false

```tsx
  const [state, setState] = useDelayedState(1000)

  setState(true)
  state -> true

  setState(false)
  setTimeout(() => {
    state -> false
  }, 1000)
```

```tsx
const useDelayedState = <T,>(initialState: T, initialDelay?: number): [T, (state: T, delay?: number) => void] => {
  const [state, setState] = useState(initialState)

  const handleStateDelay = (state: T, delay?: number) => {
    if (state) {
      setState(state)
    } else {
      setTimeout(() => {
        setState(state)
      }, delay  initialDelay  1000)
    }
  }

  return [state, handleStateDelay]
}

export default useDelayedState
```

Page example

```tsx
const MyCareerCareerPathsPage: FC<IMyCareerCareerPathsPageProps> = () => {
  // A hook with a boolean state and 2 handlers to control hint panels
  const {
    handleCloseHint: handleCloseHintPanel,
    handleOpenHint: handleOpenHintPanel,
    showHintPanel,
  } = useShowHint("showCareerPathsHintPanel");

  // Text for hint panel button
  const skillbeeButtonTitle = showHintPanel ? "Hide hint" : "Show hint";

  // Handler to toggle the hint panel button
  const handleClickSkillbeeButton = showHintPanel
    ? handleCloseHintPanel
    : handleOpenHintPanel;

  return (
    // Query component which fetches data, shows a loader if the query is pending, or shows an error message
    <Query className={"p-2"} query={api.hardSkills.pathScheme.employeeCard.get}>
      {(employeeInfo, { isLoading }) => {
        const { HasNoEvaluation, Employee, OrgUnitsHierarchy } = employeeInfo;
        const { FunctionalProfile } = employeeInfo;
        const { Name } = FunctionalProfile;

        return (
          <section className={"pb-16"}>
            {/* Button to show/hide hint panel */}
            <SkillbeeSection.Button
              className={"mt-2 mb-8 ml-auto"}
              title={skillbeeButtonTitle}
              onClick={handleClickSkillbeeButton}
            />

            {/* Hint panel */}
            <CareerPaths.HintPanel
              show={showHintPanel}
              onCloseHintPanel={handleCloseHintPanel}
            />
            {/* A panel with employee info */}
            <InfoSection.EmployeeCard
              employee={Employee}
              isLoading={isLoading}
              orgUnitsHierarchy={OrgUnitsHierarchy}
            />

            <Text.H2 className={"mt-10 mb-10"}>Current profile: {Name}</Text.H2>
            {/* A message if no career paths were found */}
            {HasNoEvaluation && <CareerPaths.EmptyPath className={"mb-12"} />}

            {/* Section with favourite career paths */}
            <Claims claims={["CareerPathFavorites"]}>
              <CareerPaths.FavoritePath />
            </Claims>

            {/* Section with expert career paths */}
            <Claims claims={["CareerPathExpert"]}>
              <CareerPaths.ExpertPath />
            </Claims>
            {/* Section with non-expert career paths */}
            <Claims claims={["CareerPathInterRole"]}>
              <CareerPaths.MixedPath />
            </Claims>
          </section>
        );
      }}
    </Query>
  );
};

MyCareerCareerPathsPage.displayName = "MyCareerCareerPathsPage";

export default MyCareerCareerPathsPage;
```
